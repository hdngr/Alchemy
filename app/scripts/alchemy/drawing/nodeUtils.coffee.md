    Alchemy.drawing.NodeUtils =
            nodeStyle: (d) ->
                conf = Alchemy.conf          
                if conf.cluster
                    nodeColours = do (d)->
                        clustering = Alchemy.layout._clustering
                        node = Alchemy._nodes[d.id].getProperties()
                        clusterMap = clustering.clusterMap
                        key = Alchemy.conf.clusterKey
                        colours = conf.clusterColours
                        # Modulo makes sure to reuse colors if it runs out
                        colourIndex = clusterMap[node[key]] % colours.length
                        colour = colours[colourIndex]
                        "#{colour}"
                else
                    nodeColours = -> if conf.nodeColour then conf.nodeColour else ''
                d

            nodeText: (d) ->
                conf = Alchemy.conf
                nodeProps = Alchemy._nodes[d.id]._properties
                if conf.nodeCaption and typeof conf.nodeCaption is 'string'
                    if nodeProps[conf.nodeCaption]?
                        nodeProps[conf.nodeCaption]
                    else
                        ''
                else if conf.nodeCaption and typeof conf.nodeCaption is 'function'
                    caption = conf.nodeCaption(nodeProps)
                    if caption is undefined or String(caption) is 'undefined'
                        Alchemy.log["caption"] = "At least one caption returned undefined"
                        conf.caption = false
                    caption
