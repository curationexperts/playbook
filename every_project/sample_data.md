# Sample Data

While the types of data vary between projects, all projects need some set of data that accurately represents the final objects we are building. During development and acceptance testing, we want it available with as little fuss as possible. We want to know where to find it, want it to be easy to use, and we need to know that we won't break anything by using it.

And so our rules for acceptable data are:

  1. It has been sanitized and approved for our use
  2. It's been taken from production records so we can check our work against them
  3. We have read and write access to it (if we're working in a hosted environment)

  Our standard server path is `/opt/data/`, and we access it with `ENV['SAMPLE_DATA_DIR']`.
