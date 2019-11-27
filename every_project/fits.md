# FITS setup

We use [FITS](https://projects.iq.harvard.edu/fits/home) for file characterization. Hyrax expects FITS to run as a standalone command-line service, but we have discovered a significant performance improvement can be gained by running FITS as a servlet and configuring Hyrax accordingly.

## 1. Install FITS Servlet

1. Install FITS servlet on the system where you'll run Hyrax. We have an ansible script to do this, here: https://github.com/curationexperts/ansible-samvera/tree/master/roles/fits
1. Ensure FITS is running and you can reach it from the system in question:

  ```bash
  ubuntu@cyp18:~$ curl http://localhost:8080/fits-1.2.0/ | grep FITS
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<title>FITS Service</title>
      <h1>FITS Web Service</h1>
  ```

## 2. Configure Hyrax to use FITS servlet

1. Add the servlet address to `.env.production`, possibly via an ansible build script. The line in `.env.production` should look like this (whatever URL you ensured was working in the above step):

  ```
  FITS_SERVLET_URL=http://localhost:8080/fits-1.2.0/
  ```

1. Ensure the `hydra-file_characterization` gem is version 1.1 or greater. Specify something like this in your `Gemfile`:

  ```
  gem 'hydra-file_characterization', '~> 1.1'
 ```

1. Define a class to determine if a file is a valid image, and of course it should also have tests.

  ```ruby
  # frozen_string_literal: true
  require 'rails_helper'

  RSpec.describe Cypripedium::ImageValidator do
    let(:corrupted_file) { Rails.root.join('spec', 'fixtures', 'images', 'corrupted', 'corrupt.png') }
    let(:good_file) { Rails.root.join('spec', 'fixtures', 'images', 'good', 'watermelon.png') }
    let(:image_validator_with_corrupted_file) { described_class.new(image: corrupted_file) }
    let(:image_validator) { described_class.new(image: good_file) }

    describe '#valid?' do
      it 'returns false if the image is currupted' do
        expect(image_validator_with_corrupted_file.valid?).to eq(false)
      end

      it 'returns true if the image is valid' do
        expect(image_validator.valid?).to eq(true)
      end
    end
  end
  ```

  ```ruby
    # frozen_string_literal: true

    module Cypripedium
      class ImageValidator
        def initialize(image:)
          @image = image
        end

        def valid?
          MiniMagick::Image.new(@image).identify
          true
        rescue MiniMagick::Error
          false
        end
      end
    end
  ```

1. Make a characterization service in `config/initializers/characterization_service.rb`:

  ```ruby
  # frozen_string_literal: true
  require 'hydra-file_characterization'
  require 'nokogiri'

  module Hydra::Works
    class CharacterizationService
      # @param [Hydra::PCDM::File] object which has properties to recieve characterization values.
      # @param [String, File] source for characterization to be run on.  File object or path on disk.
      #   If none is provided, it will assume the binary content already present on the object.
      # @param [Hash] options to be passed to characterization.  parser_mapping:, parser_class:, tools:
      def self.run(object, source = nil, options = {})
        new(object, source, options).characterize
      end

      attr_accessor :object, :source, :mapping, :parser_class, :tools

      def initialize(object, source, options)
        @object       = object
        @source       = source
        @mapping      = options.fetch(:parser_mapping, Hydra::Works::Characterization.mapper)
        @parser_class = options.fetch(:parser_class, Hydra::Works::Characterization::FitsDocument)
      end

      # Get given source into form that can be passed to Hydra::FileCharacterization
      # Use Hydra::FileCharacterization to extract metadata (an OM XML document)
      # Get the terms (and their values) from the extracted metadata
      # Assign the values of the terms to the properties of the object
      def characterize
        content = source_to_content
        extracted_md = extract_metadata(content)
        terms = parse_metadata(extracted_md)
        store_metadata(terms)
      end

      protected

        # @return content of object if source is nil; otherwise, return a File or the source
        def source_to_content
          return object.content if source.nil?
          # do not read the file into memory It could be huge...
          return File.open(source) if source.is_a? String
          source.rewind
          source.read
        end

        def extract_metadata(content)
          if ENV['FITS_SERVLET_URL']
            Hydra::FileCharacterization.characterize(content, file_name, :fits_servlet)
          else
            Hydra::FileCharacterization.characterize(content, file_name, :fits) do |cfg|
              cfg[:fits] = Hydra::Derivatives.fits_path
            end
          end
        end

        # Determine the filename to send to Hydra::FileCharacterization. If no source is present,
        # use the name of the file from the object; otherwise, use the supplied source.
        def file_name
          if source
            source.is_a?(File) ? File.basename(source.path) : File.basename(source)
          else
            object.original_name.nil? ? "original_file" : object.original_name
          end
        end

        # Use OM to parse metadata
        def parse_metadata(metadata)
          omdoc = parser_class.new
          omdoc.ng_xml = Nokogiri::XML(metadata) if metadata.present?
          omdoc.__cleanup__ if omdoc.respond_to? :__cleanup__
          characterization_terms(omdoc)
        end

        # Get proxy terms and values from the parser
        def characterization_terms(omdoc)
          h = {}
          omdoc.class.terminology.terms.each_pair do |key, target|
            # a key is a proxy if its target responds to proxied_term
            next unless target.respond_to? :proxied_term
            begin
              h[key] = omdoc.send(key)
            rescue NoMethodError
              next
            end
          end
          h.delete_if { |_k, v| v.empty? }
        end

        # Assign values of the instance properties from the metadata mapping :prop => val
        def store_metadata(terms)
          terms.each_pair do |term, value|
            property = property_for(term)
            next if property.nil?
            # Array-ify the value to avoid a conditional here
            Array(value).each { |v| append_property_value(property, v) }
          end
        end

        # Check parser_config then self for matching term.
        # Return property symbol or nil
        def property_for(term)
          if mapping.key?(term) && object.respond_to?(mapping[term])
            mapping[term]
          elsif object.respond_to?(term)
            term
          end
        end

        def append_property_value(property, value)
          # We don't want multiple mime_types; this overwrites each time to accept last value
          value = object.send(property) + [value] unless property == :mime_type
          # We don't want multiple heights / widths, pick the max
          value = value.map(&:to_i).max.to_s if property == :height || property == :width
          object.send("#{property}=", value)
        end
    end
  end
  ```

1. Finally, override Hyrax's CharacterizeJob to use the new CharacterizationService in `app/jobs/hyrax/characterize_job.rb`:

  ```ruby
  # frozen_string_literal: true
# Local override of the class at https://github.com/samvera/hyrax/blob/master/app/jobs/characterize_job.rb
# We are overriding this locally so we can improve and customize the error logging.
class Hyrax::CharacterizeJob < Hyrax::ApplicationJob
  queue_as Hyrax.config.ingest_queue_name

  # Characterizes the file at 'filepath' if available, otherwise, pulls a copy from the repository
  # and runs characterization on that file.
  # @param [FileSet] file_set
  # @param [String] file_id identifier for a Hydra::PCDM::File
  # @param [String, NilClass] filepath the cached file within the Hyrax.config.working_path
  def perform(file_set, file_id, filepath = nil)
    raise "#{file_set.class.characterization_proxy} was not found for FileSet #{file_set.id}" unless file_set.characterization_proxy?
    filepath = Hyrax::WorkingDirectory.find_or_retrieve(file_id, file_set.id) unless filepath && File.exist?(filepath)
    Hydra::Works::CharacterizationService.run(file_set.characterization_proxy, filepath)
    Rails.logger.debug "Ran characterization on #{file_set.characterization_proxy.id} (#{file_set.characterization_proxy.mime_type})"
    file_set.characterization_proxy.save!
    file_set.update_index
    file_set.parent&.in_collections&.each(&:update_index)

    # Check that MiniMagick agrees that this is a TIFF
    if Cypripedium::ImageValidator.new(image: filepath).valid?
      # Continue to create derivatives
      CreateDerivativesJob.perform_later(file_set, file_id, filepath)
    else
      # If there is a MiniMagick error, record an error and don't continue to
      # create derivatives for that file
      error = Cypripedium::CorruptFileError.new(file_set_id: file_set.id, mime_type: file_set.characterization_proxy.mime_type)
      Rails.logger.error error.message
    end
  end
end
  ```

1. At this point, you should be able to add a work as usual to Hyrax and watch it use the new FITS servlet instead of spinning up a command line instance of FITS for each object.
