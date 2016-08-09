# Blacklight::Hierarchy
[![Build Status](https://travis-ci.org/sul-dlss/blacklight-hierarchy.svg?branch=master)](https://travis-ci.org/sul-dlss/blacklight-hierarchy) [![Coverage Status](https://coveralls.io/repos/sul-dlss/blacklight-hierarchy/badge.png)](https://coveralls.io/r/sul-dlss/blacklight-hierarchy) [![Dependency Status](https://gemnasium.com/sul-dlss/blacklight-hierarchy.svg)](https://gemnasium.com/sul-dlss/blacklight-hierarchy) [![Gem Version](https://badge.fury.io/rb/blacklight-hierarchy.svg)](http://badge.fury.io/rb/blacklight-hierarchy)

This plugin provides hierarchical facets for [Blacklight](https://github.com/projectblacklight/blacklight).

## Usage

Add the plugin to your Blacklight app's Gemfile

    gem 'blacklight-hierarchy'
        
Index your hierarchies in colon-separated list. For example, items in a "processing" queue with a "copy" action, might be indexed as

    <doc>
      <field name="id">foo</field>
      <field name="queue_status_facet">processing</field>
      <field name="queue_status_facet">processing:copy</field>
      <field name="queue_status_facet">processing:copy:waiting</field>
    </doc>
    <doc>
      <field name="id">bar</field>
      <field name="queue_status_facet">processing</field>
      <field name="queue_status_facet">processing:copy</field>
      <field name="queue_status_facet">processing:copy:completed</field>
    </doc>
        
That would cause the facet count to appear at all three levels

- [processing](#) (2)
    - [copy](#) (2)
        - [completed](#) (1)
        - [waiting](#) (1)

You can skip as many levels as you'd like, as long as the "leaf" values are indexed. For example, if you didn't index the "processing" part alone, it will simply be a container, not a clickable/countable facet:

- processing
    - [copy](#) (2)
        - [completed](#) (1)
        - [waiting](#) (1)

(**Note**: If you use Solr's built-in [PathHierarchyTokenizerFactory](http://wiki.apache.org/solr/AnalyzersTokenizersTokenFilters#solr.PathHierarchyTokenizerFactory), you can index the entire depth by supplying only the leaf nodes.)

In your Blacklight controller configuration (usually `CatalogController`), tell Blacklight to render the facet using the hierarchy partial

    config.add_facet_field 'queue_status_facet', :label => 'Queue Status', 
        :partial => 'blacklight/hierarchy/facet_hierarchy'


Add the hierarchy-specific options to the controller configuration

    config.facet_display = {
      :hierarchy => {
        'queue_status' => {
          fields: ['facet'],
          delimiter: ':'
        }
      }
    }

In the above configuration, 'queue_status_facet' is the full Solr field name, and ':' is the delimiter within the field.  Note that suffixes ('facet' in the above example) should not contain underscores, since the methods that deal with the Solr fields and match them to the config assume the "prefix" ('queue_status' in the above example) will be everything up to the last underscore in the field name.  See the facet_tree method for further explanation and some relevant code, as well as the render_hierarchy method for relevant code.

## Caveats

This code was ripped out of another project, and is still quite immature as a standalone project. Every effort has been made to make it as plug-and-play as possible, but it may stomp on Blacklight in unintended ways (e.g., ways that made sense in context of its former host app, but which aren't compatible with generic Blacklight). Proceed with caution, and report issues.

## TODO

- WRITE TESTS
- Switch internal facet management from hack-y Hash to `Blacklight::Hierarchy::FacetGroup` class (already implemented, but not plumbed up)
- Add configuration support for hierarchy delimiters other than `/\s*:\s*/` (baked into `Blacklight::Hierarchy::FacetGroup`, but again, requiring additional plumbing)