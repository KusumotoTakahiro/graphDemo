<template>
  <row>
    half.x: {{half.x}}
    half.y: {{half.y}}
    <v-btn @click="add_node">ノード追加</v-btn>
    <v-btn @click="removeParentsOfOneChild">クリア</v-btn>
    <div id="cy"></div>
  </row>
</template>

<script>
import cytoscape from 'cytoscape';

export default {
  name: 'IndexPage',
  data() {
    return {
      data: 'hello world',
      node1: null,
      node2: null,
      half: {x:0, y:0},
      number: 2,
      id: 'b',
      removeEmptyParents : false, //demoサイトから引用
    }
  },
  methods: {
    //気分転換に追加
    add_node() {
      let vm = this;
      //console.log(vm.autoId()); //autoId()は動作確認済み
      let id = vm.autoId();
      this.cy.add([
        {
          group : 'nodes',
          data : {
            id : id, 
          },
          // position : {
          //   x : 150 + Math.cos(this.theta)*this.k,
          //   y : 150 + Math.sin(this.theta)*this.k,
          // },
        }
      ]);
    },

    autoId() {
      const alf = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z'];
      //console.log(alf.length);
      if (this.number<26) {
        let len = (this.id).length;
        let new_id = this.id.slice(0, len-1);
        new_id += alf[this.number];
        this.id = new_id;
        this.number += 1;
      }
      else {
        this.number = 1;
        this.id += 'a';
      }
      return this.id;
    },

    draw_circle(){
      console.log('draw_circle');
      
      let circle = document.createElement("div");
      circle.setAttribute("id", "circle")
      circle.setAttribute("position", "absolute");
      circle.setAttribute("z-index", "3");
      circle.setAttribute("width", "200px");
      circle.setAttribute("height", "200px");
      circle.setAttribute("border-radius", "50%");
      circle.setAttribute("background", "conic-gradient(green 0%,green 73% ,grey 73%,grey 100%)");
      document.body.appendChild(circle);
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
          },
          {
            selector: 'node:parent',
            style: {
              'label': ''
            }
          },
          {
            selector: '.cdnd-grabbed-node',
            style: {
              'background-color': 'red'
            }
          },

          {
            selector: '.cdnd-drop-sibling',
            style: {
              'background-color': 'red'
            }
          },

          {
            selector: '.cdnd-drop-target',
            style: {
              'border-color': 'red',
              'border-style': 'dashed'
            }
          }
        ],

        layout: {
          name: 'grid',
          rows: 1
        }
      })
    },
    //demoサイトからそのまま引用
    isParentOfOneChild(node) {
      return node.isParent() && node.children().length === 1;
    },
    removeParent(parent) {
      parent.children().move({parent:null});
      parent.remove();
    },
    removeParentsOfOneChild() {
      this.cy.nodes().filter(this.isParentOfOneChild).forEach(this.removeParent);
    },
  },
  mounted() {
    let vm = this;
    const options = {
      grabbedNode: node => true, // filter function to specify which nodes are valid to grab and drop into other nodes
      dropTarget: (dropTarget, grabbedNode) => true, // filter function to specify which parent nodes are valid drop targets
      dropSibling: (dropSibling, grabbedNode) => true, // filter function to specify which orphan nodes are valid drop siblings
      newParentNode: (grabbedNode, dropSibling) => ({}), // specifies element json for parent nodes added by dropping an orphan node on another orphan (a drop sibling). You can chose to return the dropSibling in which case it becomes the parent node and will be preserved after all its children are removed.
      boundingBoxOptions: { // same as https://js.cytoscape.org/#eles.boundingBox, used when calculating if one node is dragged over another
        includeOverlays: false,
        includeLabels: true
      },
      overThreshold: 10, // make dragging over a drop target easier by expanding the hit area by this amount on all sides
      outThreshold: 10 // make dragging out of a drop target a bit harder by expanding the hit area by this amount on all sides
    };
    this.init();
    this.tap_edge();
    this.cy.fit();
    this.cy.compoundDragAndDrop(options);
    this.cy.on('remove', (event)=>{
      if (vm.removeEmptyParents ) {
        vm.removeParentsOfOneChild();
      }
    });
    this.cy.on('cdndout', (event, dropTarget) => {
      if( vm.removeEmptyParents && vm.isParentOfOneChild(dropTarget) ){
        vm.removeParent(dropTarget);
      }
    })
  }
}
</script>
<style>
#cy {
  background-color: #f3f3f2;
  height: calc(100vw/3);
}

</style>
