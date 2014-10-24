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
    Alchemy.startGraph = (data) =>
        conf = Alchemy.conf

        if d3.select(conf.divSelector).empty()
            console.warn Alchemy.utils.warnings.divWarning()

        # see if data is ok
        if not data
            Alchemy.utils.warnings.dataWarning()

        # create nodes map and update links
        Alchemy.create.nodes.apply @, data.nodes

        data.edges.forEach (e) ->
            Alchemy.create.edges e

        # create SVG
        Alchemy.vis = d3.select conf.divSelector
            .attr "style", "width:#{conf.graphWidth()}px; height:#{conf.graphHeight()}px; background:#{conf.backgroundColour}"
            .append "svg"
                .attr "xmlns", "http://www.w3.org/2000/svg"
                .attr "xlink", "http://www.w3.org/1999/xlink"
                .attr "pointer-events", "all"
                .on 'click', Alchemy.interactions.deselectAll
                .call Alchemy.interactions.zoom(conf.scaleExtent)
                .on "dblclick.zoom", null
                .append 'g'
                    .attr "transform","translate(#{conf.initialTranslate}) scale(#{conf.initialScale})"

        Alchemy.generateLayout()
        Alchemy.controlDash.init()

        #enter/exit nodes/edges
        d3Edges = _.flatten _.map(Alchemy._edges, (edgeArray) -> e._d3 for e in edgeArray)
        d3Nodes = _.map Alchemy._nodes, (n) -> n._d3

        # if start
        Alchemy.layout.positionRootNodes()
        Alchemy.force.start()
        while Alchemy.force.alpha() > 0.005
            Alchemy.force.tick()

        Alchemy._drawEdges = Alchemy.drawing.DrawEdges
        Alchemy._drawEdges.createEdge d3Edges
        Alchemy._drawNodes = Alchemy.drawing.DrawNodes
        Alchemy._drawNodes.createNode d3Nodes

        initialComputationDone = true
        console.log Date() + ' completed initial computation'

        nodes = Alchemy.vis.selectAll 'g.node'
                        .attr 'transform', (id, i) -> "translate(#{id.x}, #{id.y})"

        # configuration for forceLocked
        if !conf.forceLocked
            Alchemy.force
                    .on "tick", Alchemy.layout.tick
                    .start()

        # call user-specified functions after load function if specified
        # deprecate?
        if conf.afterLoad?
            if typeof conf.afterLoad is 'function'
                conf.afterLoad()
            else if typeof conf.afterLoad is 'string'
                Alchemy[conf.afterLoad] = true

        if conf.initialScale isnt Alchemy.defaults.initialScale
            Alchemy.interactions.zoom().scale conf.initialScale
            return

        if conf.initialTranslate isnt Alchemy.defaults.initialTranslate
            Alchemy.interactions.zoom().translate conf.initialTranslate
            return

        if conf.cluster or conf.directedEdges
            defs = d3.select("#{Alchemy.conf.divSelector} svg").append "svg:defs"

        if conf.directedEdges
            arrowSize = conf.edgeArrowSize + (conf.edgeWidth() * 2)
            marker = defs.append "svg:marker"
                .attr "id", "arrow"
                .attr "viewBox", "0 -#{arrowSize * 0.4} #{arrowSize} #{arrowSize}"
                .attr 'markerUnits', 'userSpaceOnUse'
                .attr "markerWidth", arrowSize
                .attr "markerHeight", arrowSize
                .attr "orient", "auto"
            marker.append "svg:path"
                .attr "d", "M #{arrowSize},0 L 0,#{arrowSize * 0.4} L 0,-#{arrowSize * 0.4}"
            if conf.curvedEdges
                marker.attr "refX", arrowSize + 1
            else
                marker.attr 'refX', 1

        if conf.showEditor
            editor = new Alchemy.editor.Editor
            editorInteractions = new Alchemy.editor.Interactions
            d3.select "body"
                .on 'keydown', editorInteractions.deleteSelected

            editor.startEditor()
    debugger