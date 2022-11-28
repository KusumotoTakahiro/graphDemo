<template>
  <row>
    half.x: {{half.x}}
    half.y: {{half.y}}
    <v-btn @click="add_node">ノード追加</v-btn>
    <div id="cy"></div>
  </row>
</template>

<script>
import cytoscape from 'cytoscape';
import { resolve } from 'path';

export default {
  name: 'IndexPage',
  data() {
    return {
      data: 'hello world',
      node1: null,
      node2: null,
      half: {x:0, y:0},
      number: 1,
    }
  },
  methods: {
    //気分転換に追加
    add_node() {
      let vm = this;
      this.cy.add([
        {
          group: 'nodes',
          date: {
            id: vm.number,
          }
        }
      ])
      vm.number += 1;
    },

    draw_circle(){
      console.log('draw_circle');
    },

    get_half() {
      console.log('get_half')
      let node1 = this.cy.getElementById(this.node1).renderedPosition();
      let node2 = this.cy.getElementById(this.node2).renderedPosition();
      console.log(node1)
      console.log(node2)
      this.half = {
        x: (node1.x+node2.x)/2,
        y: (node1.y+node2.y)/2,
      }
      console.log(this.half)
    },
    tap_edge() {
      console.log('get_edge_info')
      let vm = this;
      this.cy.on('tap', 'edge', function(evt) {
        //cy.onの中では，同期処理になるみたい
        console.log('tap!!')
        const data = evt.target._private.data;
        vm.node1 = evt.target._private.data.source;
        vm.node2 = evt.target._private.data.target;
        //console.log(evt);
        vm.get_half();
        vm.draw_circle();
      })
    },
    init() {
      this.cy = cytoscape({
        container : document.getElementById('cy'),
        elements: [ // list of graph elements to start with
          { // node a
            data: { id: 'a' }
          },
          { // node b
            data: { id: 'b' }
          },
          { // edge ab
            data: { id: 'ab', source: 'a', target: 'b' }
          }
        ],

        style: [ // the stylesheet for the graph
          {
            selector: 'node',
            style: {
              'background-color': '#666',
              'label': 'data(id)'
            }
          },

          {
            selector: 'edge',
            style: {
              'width': 3,
              'line-color': '#ccc',
              'target-arrow-color': '#ccc',
              'target-arrow-shape': 'triangle',
              'curve-style': 'bezier',
              'arrow-scale': '3',
            }
          }
        ],

        layout: {
          name: 'grid',
          rows: 1
        }
      })
    }
  },
  mounted() {
    this.init();
    this.tap_edge();
    this.cy.fit();
  }
}
</script>
<style>
#cy {
  background-color: #f3f3f2;
  height: calc(100vw/3);
}
</style>
