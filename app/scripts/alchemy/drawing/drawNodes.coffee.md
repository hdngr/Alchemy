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

    Alchemy.drawing.DrawNodes =
        createNode: (d3Nodes) ->
            drawNode = Alchemy.drawing.DrawNode

            # AlchemyNode is an array of one or more AlchemyNode._d3 packets
            node = Alchemy.vis.selectAll "g.node"
                            .data d3Nodes, (n) -> n.id
            node.enter().append "g"
                    .attr "class", (d) ->
                        nodeType = Alchemy._nodes[d.id]._nodeType
                        "node #{nodeType} active"
                    .attr 'id', (d) -> "node-#{d.id}"
                    .classed 'root', (d) -> d.root

            drawNode.createNode node
            drawNode.styleNode node
            drawNode.styleText node
            drawNode.setInteractions node
            node.exit().remove()

        updateNode: (AlchemyNode) ->
            # AlchemyNode is an array of one or more AlchemyNode._d3 packets
            drawNode = Alchemy.drawing.DrawNode
            node = Alchemy.vis.select "#node-#{AlchemyNode.id}"
            drawNode.styleNode node
            drawNode.styleText node
            drawNode.setInteractions node

