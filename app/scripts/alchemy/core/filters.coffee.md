    # Alchemy.js is a graph drawing application for the web.
    # Copyright (C) 2014  GraphAlchemist, Inc.

    # This program is free software: you can redistribute it and/or modify
    # it under the terms of the GNU Affero General Public License as published by
    # the Free Software Foundation, either version 3 of the License, or
    # (at your option) any later version.

    # This program is distributed in the hope that it will be useful,
    # but WITHOUT ANY WARRANTY; without even the implied warranty of
    # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    # GNU Affero General Public License for more details.

    # You should have received a copy of the GNU Affero General Public License
    # along with this program.  If not, see <http://www.gnu.org/licenses/>.
    Alchemy.filters = 
        init: () -> 
            Alchemy.filters.show()
            
            if Alchemy.conf.edgeFilters then Alchemy.filters.showEdgeFilters()
            if Alchemy.conf.nodeFilters then Alchemy.filters.showNodeFilters()
            #generate filter forms
            if Alchemy.conf.nodeTypes
                nodeKey = Object.keys Alchemy.conf.nodeTypes

                nodeTypes = ''
                for nodeType in Alchemy.conf.nodeTypes[nodeKey]
                    # Create Filter list element
                    caption = nodeType.replace '_', ' '
                    nodeTypes += "<li class='list-group-item nodeType' role='menuitem' id='li-#{nodeType}' name=#{nodeType}>#{caption}</li>"
                Alchemy.dash.select '#node-dropdown'
                       .html nodeTypes

            if Alchemy.conf.edgeTypes
                for e in Alchemy.dash.selectAll(".edge")[0]
                    Alchemy.currentRelationshipTypes[[e].caption] = true

                edgeTypes = ''
                for edgeType in Alchemy.conf.edgeTypes
                    # Create Filter list element
                    caption = edgeType.replace '_', ' '
                    edgeTypes += "<li class='list-group-item edgeType' role='menuitem' id='li-#{edgeType}' name=#{edgeType}>#{caption}</li>"
                Alchemy.dash.select '#rel-dropdown'
                       .html edgeTypes
            
            if Alchemy.conf.captionsToggle then Alchemy.filters.captionsToggle()
            if Alchemy.conf.edgesToggle then Alchemy.filters.edgesToggle()
            if Alchemy.conf.nodesToggle then Alchemy.filters.nodesToggle()
            Alchemy.filters.update()

        show: ->
            filter_html = """
                        <div id = "filter-header" data-toggle="collapse" data-target="#filters form">
                            <h3>Filters</h3>
                            <span class = "fa fa-2x fa-caret-right"></span>
                        </div>
                            <form class="form-inline collapse">
                            </form>
                          """
            Alchemy.dash.select('#control-dash #filters').html filter_html
            Alchemy.dash.selectAll '#filter-header'
                .on 'click', () ->
                    if Alchemy.dash.select('#filters>form').classed "in"
                        Alchemy.dash.select "#filter-header>span"
                               .attr "class", "fa fa-2x fa-caret-right"
                    else
                        Alchemy.dash.select "#filter-header>span"
                               .attr "class", "fa fa-2x fa-caret-down"

            Alchemy.dash.select '#filters form'
                   # .submit false

        #create relationship filters
        showEdgeFilters: () ->
            rel_filter_html = """
                            <div id="filter-rel-header" data-target = "#rel-dropdown" data-toggle="collapse">
                                <h4>
                                    Edge Types
                                </h4>
                                <span class="fa fa-lg fa-caret-right"></span>
                            </div>
                            <ul id="rel-dropdown" class="collapse list-group" role="menu">
                            </ul>
                               """
            Alchemy.dash.select '#filters form'
                   .append "div"
                   .attr "id", "filter-relationships"
                   .html rel_filter_html
            Alchemy.dash.select "#filter-rel-header"
                .on 'click', () ->
                    if Alchemy.dash.select('#rel-dropdown').classed "in"
                        Alchemy.dash.select "#filter-rel-header>span"
                               .attr "class", "fa fa-lg fa-caret-right"
                    else
                        Alchemy.dash.select "#filter-rel-header>span"
                               .attr "class", "fa fa-lg fa-caret-down"

        #create node filters
        showNodeFilters: () ->
            node_filter_html = """
                                <div id="filter-node-header" data-target = "#node-dropdown" data-toggle="collapse">
                                    <h4>
                                        Node Types
                                    </h4>
                                    <span class="fa fa-lg fa-caret-right"></span>
                                </div>
                                <ul id="node-dropdown" class="collapse list-group" role="menu">
                                </ul>
                               """
            Alchemy.dash.select '#filters form'
                   .append "div"
                   .attr "id", "filter-nodes"
                   .html node_filter_html
            Alchemy.dash.select "#filter-node-header"    
                .on 'click', () ->
                    if Alchemy.dash.select('#node-dropdown').classed "in"
                        Alchemy.dash.select "#filter-node-header>span"
                               .attr "class", "fa fa-lg fa-caret-right"
                    else 
                        Alchemy.dash.select "#filter-node-header>span"
                               .attr "class", "fa fa-lg fa-caret-down"

        #create captions toggle
        captionsToggle: () ->
            Alchemy.dash.select "#filters form"
              .append "li"
              .attr {"id":"toggle-captions","class":"list-group-item active-label toggle"}
              .html "Show Captions"
              .on "click", ->
                isDisplayed = Alchemy.dash.select("g text").attr("style")

                if isDisplayed is "display: block" || null
                    Alchemy.dash.selectAll "g text"
                           .attr "style", "display: none"
                else
                    Alchemy.dash.selectAll "g text"
                           .attr "style", "display: block"

        #create edges toggle
        edgesToggle: () ->
            Alchemy.dash.select "#filters form"
              .append "li"
              .attr {"id":"toggle-edges","class":"list-group-item active-label toggle"}
              .html "Toggle Edges"
              .on "click", ->
                  _.each _.values(Alchemy._edges), (edges)->
                      _.each edges, (e)-> e.toggleHidden()

        #create nodes toggle
        nodesToggle: () ->
            Alchemy.dash.select "#filters form"
              .append "li"
              .attr {"id":"toggle-nodes","class":"list-group-item active-label toggle"}
              .html "Toggle Nodes"
              .on "click", ->
                  _.each _.values(Alchemy._nodes), (n)->
                      if Alchemy.conf.toggleRootNodes and n._d3.root then return
                      n.toggleHidden()

        #update filters
        update: () ->
            Alchemy.dash.selectAll ".nodeType, .edgeType"
                .on "click", () ->
                    element = d3.select this
                    tag = element.attr "name"
                    Alchemy.vis.selectAll ".#{tag}"
                        .each (d)-> 
                            if Alchemy._nodes[d.id]?
                                node = Alchemy._nodes[d.id]
                                node.toggleHidden()
                            else
                                edge = Alchemy._edges[d.id][0]
                                edge.toggleHidden()