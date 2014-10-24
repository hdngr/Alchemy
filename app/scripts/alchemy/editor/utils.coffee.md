    class Alchemy.editor.Utils
        constructor: () ->
            @drawNodes = Alchemy._drawNodes
            @drawEdges = Alchemy._drawEdges

        enableEditor: () =>
            Alchemy.set.state "interactions", "editor"
            dragLine = Alchemy.vis
                .append "line"
                .attr "id", "dragline"

            @drawNodes.updateNode Alchemy.node
            @drawEdges.updateEdge Alchemy.edge
            selectedElements = Alchemy.vis.selectAll ".selected"
            editor = new Alchemy.editor.Editor
            if (not selectedElements.empty()) and (selectedElements.length is 1)
                if selectedElements.classed 'node'
                    editor.nodeEditor selectedElements.datum()
                    Alchemy.dash
                        .select "#node-editor" 
                        .attr "class", "enabled"
                        .style "opacity", 1
                else if selectedElements.classed 'edge'
                    editor.edgeEditor selectedElements.datum()
                    Alchemy.dash
                        .select "#edge-editor"
                        .attr "class", "enabled"
                        .style "opacity", 1
            else
                selectedElements.classed "selected":false

        disableEditor: () ->
            Alchemy.setState "interactions", "default"
            Alchemy.vis
                   .select "#dragline"
                   .remove()

            Alchemy.dash
                   .select "#node-editor"
                   .transition()
                   .duration 300
                   .style "opacity", 0
            Alchemy.dash
                   .select "#node-editor"
                   .transition()
                   .delay 300
                   .attr "class", "hidden"

            @drawNodes.updateNode Alchemy.node
            Alchemy.vis
                   .selectAll ".node"
                   .classed "selected":false

        remove: () ->
            selectedNodes = Alchemy.vis.selectAll ".selected.node"
            for node in selectedNodes[0]
                nodeID = Alchemy.vis
                                .select node
                                .data()[0]
                                .id

                node_data = Alchemy._nodes[nodeID]
                if node_data?  
                    for edge in node_data.adjacentEdges
                        Alchemy._edges = _.omit Alchemy._edges, "#{edge}"
                        Alchemy.edge = Alchemy.edge.data _.map(Alchemy._edges, (e) -> e._d3), (e)->e.id
                        Alchemy.vis
                               .select "#edge-#{edge}"
                               .remove()
                    Alchemy._nodes = _.omit Alchemy._nodes, "#{nodeID}"
                    Alchemy.node = Alchemy.node.data _.map(Alchemy._nodes, (n) -> n._d3), (n)->n.id
                    Alchemy.vis
                           .select node
                           .remove()
                    if Alchemy.get.state("interactions") is "editor"
                        Alchemy.modifyElements.nodeEditorClear()

        addNode: (node) ->
            newNode = Alchemy._nodes[node.id] = new Alchemy.models.Node {id:"#{node.id}"}
            newNode.setProperty "caption", node.caption
            newNode.setD3Property "x", node.x
            newNode.setD3Property "y", node.y
            Alchemy.node = Alchemy.node.data _.map(Alchemy._nodes, (n) -> n._d3), (n)->n.id

        addEdge: (edge) ->
            newEdge = Alchemy._edges[edge.id] = new Alchemy.models.Edge edge
            Alchemy.edge = Alchemy.edge.data _.map(Alchemy._edges, (e) -> e._d3), (e)->e.id

        update: (node, edge) ->
            #only push the node if it didn't previously exist
            if !@mouseUpNode
                Alchemy.editor.addNode node
                Alchemy.editor.addEdge edge
                @drawEdges.createEdge Alchemy.edge
                @drawNodes.createNode Alchemy.node

            else
                Alchemy.editor.addEdge edge
                @drawEdges.createEdge Alchemy.edge

            Alchemy.layout.tick()
