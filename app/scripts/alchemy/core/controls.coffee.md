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

    Alchemy.controlDash = 
        init: ->
            if @dashIsShown()
                divSelector = Alchemy.conf.divSelector
                # add dashboard wrapper
                Alchemy.dash = d3.select "#{divSelector}"
                                 .append "div"
                                 .attr "id", "control-dash-wrapper"
                                 .attr "class", "col-md-4 initial"

                # add the dash toggle button 
                Alchemy.dash
                       .append "i"
                       .attr "id", "dash-toggle"
                       .attr "class", "fa fa-flask col-md-offset-12"

                # add the control dash
                Alchemy.dash
                       .append "div"
                       .attr "id", "control-dash"
                       .attr "class", "col-md-12"

                Alchemy.dash.select '#dash-toggle'
                       .on 'click', Alchemy.interactions.toggleControlDash

                Alchemy.controlDash.zoomCtrl()
                Alchemy.controlDash.search()
                Alchemy.controlDash.filters()
                Alchemy.controlDash.stats()
                Alchemy.controlDash.clustering()

        search: ->
            if Alchemy.conf.search
                Alchemy.dash
                       .select "#control-dash"
                       .append "div"
                       .attr "id", "search"
                       .html """
                            <div class='input-group'>
                                <input class='form-control' placeholder='Search'>
                                <i class='input-group-addon search-icon'><span class='fa fa-search fa-1x'></span></i>
                            </div> 
                              """
                Alchemy.search.init()
        
        zoomCtrl: ->
            if Alchemy.conf.zoomControls 
                Alchemy.dash
                    .select "#control-dash-wrapper"
                    .append "div"
                    .attr "id", "zoom-controls"
                    .attr "class", "col-md-offset-12"
                    .html "<button id='zoom-reset'  class='btn btn-defualt btn-primary'><i class='fa fa-crosshairs fa-lg'></i></button>
                            <button id='zoom-in'  class='btn btn-defualt btn-primary'><i class='fa fa-plus'></i></button>
                            <button id='zoom-out' class='btn btn-default btn-primary'><i class='fa fa-minus'></i></button>"
                
                Alchemy.dash
                       .select '#zoom-in'
                       .on "click", -> Alchemy.interactions.clickZoom 'in'
                
                Alchemy.dash
                       .select '#zoom-out'
                       .on "click", -> Alchemy.interactions.clickZoom 'out'
                
                Alchemy.dash
                       .select '#zoom-reset'
                       .on "click", -> Alchemy.interactions.clickZoom 'reset'

        filters: ->
            if Alchemy.conf.nodeFilters or Alchemy.conf.edgeFilters
                Alchemy.dash
                    .select "#control-dash"
                    .append "div"
                    .attr "id", "filters"
                Alchemy.filters.init()

        stats: ->
            if Alchemy.conf.nodeStats or Alchemy.conf.edgeStats
                stats_html = """
                        <div id = "stats-header" data-toggle="collapse" data-target="#stats #all-stats">
                        <h3>
                            Statistics
                        </h3>
                        <span class = "fa fa-caret-right fa-2x"></span>
                        </div>
                        <div id="all-stats" class="collapse">
                            <ul class = "list-group" id="node-stats"></ul>
                            <ul class = "list-group" id="rel-stats"></ul>  
                        </div>
                    """

                Alchemy.dash
                    .select "#control-dash"
                    .append "div"
                    .attr "id", "stats"
                    .html stats_html
                    .select '#stats-header'
                    .on 'click', () ->
                        if Alchemy.dash.select('#all-stats').classed "in"
                            Alchemy.dash
                                   .select "#stats-header>span"
                                   .attr "class", "fa fa-2x fa-caret-right"
                        else
                            Alchemy.dash
                                   .select "#stats-header>span"
                                   .attr "class", "fa fa-2x fa-caret-down"

                Alchemy.stats.init()

        clustering: ->
            if Alchemy.conf.clusterControl
                clusterControl_html = """
                        <div id="clustering-container">
                            <div id="cluster_control_header" data-toggle="collapse" data-target="#clustering #cluster-options">
                                 <h3>Clustering</h3>
                                <span id="cluster-arrow" class="fa fa-2x fa-caret-right"></span>
                            </div>
                        </div>
                        """
                Alchemy.dash
                    .select "#control-dash"
                    .append "div"
                    .attr "id", "clustering"
                    .html clusterControl_html
                    .select '#cluster_control_header'

                Alchemy.clusterControls.init()

        dashIsShown: ->
            conf = Alchemy.conf
            # lol - Matt is a nerd || hates 'or'
            conf.showEditor    || conf.captionToggle  || conf.toggleRootNodes ||
            conf.removeElement || conf.clusterControl || conf.nodeStats       ||
            conf.edgeStats     || conf.edgeFilters    || conf.nodeFilters     || 
            conf.edgesToggle   || conf.nodesToggle    || conf.search