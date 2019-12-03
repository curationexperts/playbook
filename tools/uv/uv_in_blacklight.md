# Installing UV in a Blacklight application

Create a system spec called `view_work_spec.rb` to make sure that the UV `iframe` is showing up
on the page:
```ruby
# frozen_string_literal: true
require 'rails_helper'

RSpec.feature "View a Work" do
  before do
    solr = Blacklight.default_index.connection
    solr.add(work_attributes)
    solr.commit
    allow(Rails.application.config).to receive(:iiif_url).and_return('https://example.com')
  end

  let(:id) { '123' }

  let(:work_attributes) do
    {
      id: id,
      has_model_ssim: ['Work'],
      title_tesim: ['The Title of my Work'],
      description_tesim: ['Description 1', 'Description 2']
    }
  end

  it 'has the uv html on the page' do
    visit solr_document_path(id)
    expect(page.html).to match(/universal-viewer-iframe/)
  end
end
```

Set the IIIF base URL in an initializer in `config/initializers/` called `iiif_url.rb`:
```ruby
# frozen_string_literal: true

Rails.application.config.iiif_url = ENV['IIIF_URL'] || ''
```


At `app/views/catalog/` create a file called `_uv.html.erb` with the following content:
```
<iframe
    class="universal-viewer-iframe"
    src="<%= request&.base_url %>/uv/uv.html#?manifest=<%= Rails.application.config.iiif_url %>/<%= document.id %>/manifest"
    width="640"
    height="480"
    allowfullscreen
    frameborder="0">
</iframe>
</br>
```

The `src` attribute is the path to the manifest for the work/item.

In `controllers/catalog_controller.rb` add this partial to the config around line 50:
```
  config.show.partials.insert(1, :uv)
```

This will insert the uv partial in the first position on the show page. You are able to add additional partials
if necessary and change the order based on the insert number.

Your feature test should pass at this point, but there are additional things that need to be added.

Create another system spec called `view_uv_spec.rb`:
```
# frozen_string_literal: true
require 'rails_helper'
include Warden::Test::Helpers

RSpec.describe 'viewing uv', type: :system do
  it 'has expected text' do
    visit "/uv/uv.html"
    expect(page.html).to match(/uv/)
  end
end
```
This will test that UV has actually been installed.

Create a file in the root of the project called `package.json`. If this file already exists, make sure it has these dependencies and scripts:
```
{
  "name": "lux_bl_7",
  "private": true,
  "dependencies": {
    "universalviewer": "^3.0.16"
  },
  "scripts": {
    "preinstall": "rm -rf ./public/uv",
    "postinstall": "yarn run uv-install && yarn run uv-config",
    "uv-install": "shx cp -r ./node_modules/universalviewer/uv ./public/",
    "uv-config": "shx cp ./config/uv/uv.html ./public/uv/uv.html & shx cp ./config/uv/uv-config.json ./public/uv/"
  },
  "devDependencies": {
    "shx": "^0.3.2"
  }
}
```

This will use the post-install and pre-install hooks to download and install UV when running `yarn install`. `yarn install` is called during `rake assets::precompile` so it will be called when deploying the application.

Before running `yarn install`, create a folder called `uv` in the `config` folder. In that folder create two files: `uv.html` and `uv-config.json`. These files would be overwritten by the install process, but are copied into UV via the scripts in `package.json` so customizations can be made.

`uv.html`:
```html
<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <!--
        This is what the embed iframe src links to. It doesn't need to communicate with the parent page, only fill the available space and look for #? parameters
      -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
    <link rel="icon" href="favicon.ico">
    <link rel="stylesheet" type="text/css" href="uv.css">
    <script type="text/javascript" src="lib/offline.js"></script>
    <script type="text/javascript" src="helpers.js"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
        }
        .uv .mobileFooterPanel .options .btn.fullScreen {
          display: inline !important;
         }
         .title > span {
            display: none;
         }
    </style>
    <script type="text/javascript">
    window.addEventListener('uvLoaded', function(e) {
            urlDataProvider = new UV.URLDataProvider(true);
            var formattedLocales;
            var locales = urlDataProvider.get('locales', '');
            if (locales) {
                var names = locales.split(',');
                formattedLocales = [];
                for (var i in names) {
                    var nameparts = String(names[i]).split(':');
                    formattedLocales[i] = {name: nameparts[0], label: nameparts[1]};
                }
            } else {
                formattedLocales = [
                    {
                        name: 'en-GB'
                    }
                ]
            }
            uv = createUV('#uv', {
                root: '.',
                iiifResourceUri: urlDataProvider.get('manifest'),
                configUri: 'uv-config.json',
                collectionIndex: Number(urlDataProvider.get('c', 0)),
                manifestIndex: Number(urlDataProvider.get('m', 0)),
                sequenceIndex: Number(urlDataProvider.get('s', 0)),
                canvasIndex: Number(urlDataProvider.get('cv', 0)),
                rangeId: urlDataProvider.get('rid', 0),
                rotation: Number(urlDataProvider.get('r', 0)),
                xywh: urlDataProvider.get('xywh', ''),
                embedded: true,
                locales: formattedLocales
            }, urlDataProvider);
        }, false);
    </script>
</head>
<body>

<div id="uv" class="uv"></div>

<script>
    $(function() {
        var $UV = $('#uv');
        function resize() {
            var windowWidth = window.innerWidth;
            var windowHeight = window.innerHeight;
            $UV.width(windowWidth);
            $UV.height(windowHeight);
        }
        $(window).on('resize' ,function() {
            resize();
        });
        resize();
    });
</script>

<script type="text/javascript" src="uv.js"></script>

</body>
</html>
```

`uv-config.json`:

```json
{
    "options": {
        "rightPanelEnabled": true
    },
    "modules": {
        "footerPanel": {
            "options": {
                "downloadEnabled": true,
                "fullscreenEnabled": true,
                "shareEnabled": true
            }
        }
    }
}

```

Now run `yarn install` and it should succeed. Run your other system spec and it should also pass. UV is now installed
and will appear on item show pages. If their is a black square on the page, UV was not able to access a IIIF manifest
at the configured URL. If you haven't configured a IIIF url with an env variable it will try to access a manifest at a path like this `http://localhost/123/manifest` where `123` is the ID of the item.
