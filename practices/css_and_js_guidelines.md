# CSS
## Common Pattern for Hyrax applications
- Remove `require_tree .` from `application.css`
- Create a `projectsname.scss` file in `/app/assets/stylesheets`
- Create a folder for the project: `/app/assets/stylesheets/projectname`
- In `/app/assets/stylesheets/projectname/` create SASS partials `_something.scss`
- Include those partials in `projectname.scss`:

```sass
@import 'something';
```

## Essential CSS Guidelines
### Don't use camelCase or under_scores in CSS class names
- <https://cssguidelin.es/#hyphen-delimited>

### Avoid using IDs in CSS like the plague
-  <https://cssguidelin.es/#ids-in-css>

### Use lots of classes in markup instead of `nav > ul > li > a`
- <https://cssguidelin.es/#misconceptions>
- <https://cssguidelin.es/#javascript-hooks>

# JavaScript
- Use standard.js (<https://standardjs.com/>)
  - The most important parts:
    - Two-space indent
    - No semicolons
    - camelCase
    - Use single-quotes
    - No unused variables
- Have a Jasmine, Capybara feature test, or both for additional JS functionality
- Have a global object that is used as a namespace for the project, here's an example from
[Tufts](https://github.com/curationexperts/epigaea/blob/master/app/assets/javascripts/tufts.js). The Hyrax object in object is also used this way.
