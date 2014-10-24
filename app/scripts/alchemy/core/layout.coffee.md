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

    class Alchemy.Layout
        constructor: ->
            conf = Alchemy.conf
            nodes = Alchemy._nodes
            @k = Math.sqrt Math.log(_.size(Alchemy._nodes)) / (conf.graphWidth() * conf.graphHeight())
            @_clustering = new Alchemy.clustering

            # Set up quad tree
            if conf.collisionDetection
                @d3NodeInternals = _.map Alchemy._nodes, (v,k)-> v._d3
                @q = d3.geom.quadtree @d3NodeInternals
            
            if conf.cluster
                @_charge = () -> @_clustering.layout.charge
                @_linkStrength = (edge) -> @_clustering.layout.linkStrength(edge)
            else
                @_charge = () -> -10 / @k
                @_linkStrength = (edge) ->
                    if nodes[edge.source.id].getProperties('root') or nodes[edge.target.id].getProperties('root')
                        1
                    else
                        0.9

            if conf.cluster
                @_linkDistancefn = (edge) -> @_clustering.layout.linkDistancefn(edge)
            else if conf.linkDistancefn is 'default'
                @_linkDistancefn = (edge) ->
                    1 / (@k * 50)
            else if typeof conf.linkDistancefn is 'number'
                @_linkDistancefn = (edge) -> conf.linkDistancefn
            else if typeof conf.linkDistancefn is 'function'
                @_linkDistancefn = (edge) -> conf.linkDistancefn(edge)

        gravity: () =>
            if Alchemy.conf.cluster
                @_clustering.layout.gravity @k
            else
                50 * @k

        linkStrength: (edge) =>
            @_linkStrength edge

        friction: () ->
            if Alchemy.conf.cluster then 0.7 else 0.9

        collide: (node) =>
            conf = Alchemy.conf
            r = 2 * (node.radius + node['stroke-width']) + conf.nodeOverlap
            nx1 = node.x - r
            nx2 = node.x + r
            ny1 = node.y - r
            ny2 = node.y + r
            return (quad, x1, y1, x2, y2) ->
                if quad.point and (quad.point isnt node)
                    x = node.x - Math.abs quad.point.x
                    y = node.y - quad.point.y
                    l = Math.sqrt(x * x + y * y)
                    r = r
                    if l < r
                        l = (l - r) / l * Alchemy.conf.alpha
                        node.x -= x *= l
                        node.y -= y *= l
                        quad.point.x += x
                        quad.point.y += y
                x1 > nx2 or
                x2 < nx1 or
                y1 > ny2 or
                y2 < ny1

        tick: () =>
            if Alchemy.conf.collisionDetection
                for node in @d3NodeInternals
                    @q.visit @collide(node)

            # Alchemy.node
            Alchemy.vis
                .selectAll "g.node"
                .attr "transform", (d) -> "translate(#{d.x},#{d.y})"

            edges = Alchemy.vis.selectAll "g.edge"
            @drawEdge = Alchemy.drawing.DrawEdge
            @drawEdge.styleText edges
            @drawEdge.styleLink edges

        positionRootNodes: () ->
            conf = Alchemy.conf
            container =
                width: conf.graphWidth()
                height: conf.graphHeight()

            rootNodes = _.filter Alchemy.get.allNodes(), (node) -> node.getProperties('root')
            # if there is one root node, position it in the center
            if rootNodes.length is 1
                n = rootNodes[0]
                [n._d3.x, n._d3.px] = [container.width / 2, container.width / 2]
                [n._d3.y, n._d3.py] = [container.height/ 2, container.height/ 2]
                # fix root nodes until force layout is complete
                n._d3.fixed = true
                return
            # position nodes towards center of graph
            else
                for n, i in rootNodes
                    n._d3.x = container.width / Math.sqrt(rootNodes.length * (i+1))
                    n._d3.y = container.height / 2
                    n._d3.fixed = true

        chargeDistance: () ->
            500

        linkDistancefn: (edge) =>
            @_linkDistancefn edge

        charge: () ->
            @_charge()

    Alchemy.generateLayout = (start=false)->
        conf = Alchemy.conf

        Alchemy.layout = new Alchemy.Layout
        Alchemy.force = d3.layout.force()
            .size [conf.graphWidth(), conf.graphHeight()]
            .nodes _.map(Alchemy._nodes, (node) -> node._d3)
            .links _.flatten(_.map(Alchemy._edges, (edgeArray) -> e._d3 for e in edgeArray))

        Alchemy.force
            .charge Alchemy.layout.charge()
            .linkDistance (link) -> Alchemy.layout.linkDistancefn(link)
            .theta 1.0
            .gravity Alchemy.layout.gravity()
            .linkStrength (link) -> Alchemy.layout.linkStrength(link)
            .friction Alchemy.layout.friction()
            .chargeDistance Alchemy.layout.chargeDistance()
