<script>
import cytoscape from 'cytoscape';
import d3Force from 'cytoscape-d3-force';
import { onMount } from 'svelte';
import {Springy, Graph} from "../lib/springy";
import {springy } from "../lib/springyui";
onMount(() => {
    let graphData = {
				nodes: ['a', 'b', 'c', 'd', 'e', 'f', "cat", "Floof"],
				edges: [
					['a', "cat", {label:"isA"}],
					['b', "cat", {label:"isA"}],
					['c', "cat", {label:"isA"}],
					['d', "cat", {label:"isA"}],
					['e', "cat", {label:"isA"}],
					['f', "cat", {label:"isA"}],
					['a', 'b', {label:'parentOf'}],
					['a', 'c', {label: 'parentOf'}],
					['a', "Floof", {label:'hasNickname'}],
				]
			};
		
			var graph = new Graph();
			graph.loadJSON(graphData);
						
			springy(document.getElementById('springydemo'), {
			graph: graph,
			nodeSelected: function(node){
				console.log('Node selected: ' + JSON.stringify(node.data));
			}});
})

//TODO remove below

cytoscape.use( d3Force );

const group = ['hospital', 'clothes', 'computer', 'person', 'flower', 'tree', 'desk', 'house', 'water', 'cup']
const edgegroup = ['has', 'goto', 'love']
const year = ['2017', '2018', '2019']
const MAX_Y = 800
const MAX_X = 1500
function createId(salt, randomLength = 8) {
  return (
    (salt || '') +
    Number(
      Math.random()
        .toString()
        .substr(3, randomLength) + Date.now()
    ).toString(36)
  )
}
function createNodes(num) {
  let datas = []
  for (let i = 0; i < num; i++) {
    let _groupId = group[Math.floor(Math.random() * group.length)]
    let data = {
      id: createId('node_'),
      position: {
        x: Math.random() * MAX_X,
        y: Math.random() * MAX_Y,
      },
      group: _groupId,
      parent: _groupId
    }
    data.label = "Hello";//data.group + '-node' + i
    datas.push({
      group: 'nodes',
      data,
      id: data.id
    })
  }
  return datas
}
function createParent (nodes) {
  let _parents = Array.from(new Set(nodes.map(node => node.data.group))).filter(p => !nodes.find(node => node.data.id === p) && p !== 'person')
  return _parents.map(p => {
    return {
      group: 'nodes',
      data: {
        id: p,
        label: p
      },
      id: p
    }
  })
}
function createEdges(nodes, num) {
  
  let edges = []
  for (let i = 0; i < num - 1; i++) {
    let target = nodes[i + 1].data.id
    let source = nodes[Math.floor(Math.sqrt(i))].data.id
    let edge = {
      target,
      source,
      id: createId('edge_'),
      group: edgegroup[Math.floor(Math.random() * edgegroup.length)],
      time: year[Math.floor(Math.random() * year.length)] + '-' + Math.ceil(Math.random() * 12) + '-' + Math.ceil(Math.random() * 30)
    }
    edge.label = 'edge' + i
    edge.name = 'edge' + i

    edges.push({
      data: edge,
      group: 'edges',
      id: edge.id
    })
  }
  return edges
}
function createEdgesFromId(nodes, id) {
  let edges = nodes.map(node => {
    return {
      group: 'edges',
      data: {
        target: node.data.id,
        source: id,
        id: createId('edge_'),
        group: edgegroup[Math.floor(Math.random() * edgegroup.length)],
        label: node.data.id + '-' + id,
        name: node.data.id + '-' + id
      }
    }
  })
  return edges
}
function createData(num) {
  let nodes = createNodes(num)
  let edges = createEdges(nodes, num)
  return nodes.concat(edges)// .concat(createParent(nodes))
}
function createChildren(id, num) {
  let nodes = createNodes(num)
  let edges = createEdgesFromId(nodes, id)
  return nodes.concat(edges)
}

onMount(() => {

    var cy = window.cy = cytoscape({
					container: document.getElementById('cy'),

					// demo your layout
					layout: {
						name: 'd3-force',
            animate: true,
            fixedAfterDragging: false,
            linkId: function id(d) {
              return d.id;
            },
            linkDistance: 80,
            manyBodyStrength: -300,
            ready: function(){},
            stop: function(){},
            tick: function(progress) {
              console.log('progress - ', progress);
            },
            randomize: false,
            infinite: true
						// some more options here...
					},

					style: [
						// {
						// 	selector: 'node',
						// 	style: {
						// 		'content': 'data(name)'
						// 	}
						// },

						{
							selector: 'edge',
							style: {
								'curve-style': 'bezier',
								'target-arrow-shape': 'triangle'
							}
						}
					],

          elements: createData(100),
          wheelSensitivity: 0.5,
        });


			});

</script>
<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

<div id="cy"></div>

<style>
    
			/* #cy {
				position: absolute;
				left: 0;
				top: 0;
				bottom: 0;
				right: 0;
				z-index: 999;
			} */
</style>