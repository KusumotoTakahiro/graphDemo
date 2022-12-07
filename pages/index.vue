<template>
  <div>
    half.x: {{half.x}}
    half.y: {{half.y}}
    <v-card v-if="chart" class="dialog">
      <chart :chartData="chartData" :options="chartoptions"></chart>
      <v-btn @click="chart=!chart">close</v-btn>
    </v-card>
    <v-btn @click="set_data">データ変更</v-btn>
    <v-btn @click="add_node(880,300,speakers)">ノード追加</v-btn>
    <v-btn @click="removeParentsOfOneChild">クリア</v-btn>
    <div style="background-color: aliceblue; height: 100%;; " id="area"></div>
    <div id="cy"></div>
    
  </div>
</template>

<script>
import cytoscape from 'cytoscape';
import chart from '~/components/chart.vue';

export default {
  name: 'IndexPage',
  components: {chart},
  data() {
    return {
      data: 'hello world',
      a: 10,
      b: 10,
      on_edge: false,
      chart: false,
      dialog: false,
      node1: null,
      node2: null,
      half: {x:0, y:0},
      number: 2,
      id: 'b',
      removeEmptyParents : false, //demoサイトから引用
      chartData: null,
      chartoptions: {
        responsive: true,
        maintainAspectRatio: false,
      },
      speakers : [
      "19704018",
      "19704018",
      "19704012",
      "19704018",
      "19704018",
      "19704013",
      "19704012",
      "19704018",
      "19704012",
      "19704012",
      "19704012",
      "19704012",
      "17233060",
      "19704013",
      "19704013",
      "19704012",
      "19704012",
      "19704012",
      "19704013",
      "19704018"
      ]
    }
  },
  methods: {
    //気分転換に追加
    add_node(x, y, speakers) {
      let vm = this;
      let result = vm.makeSVG(this.get_prop(speakers));
      console.log(result)
      //console.log(vm.autoId()); //autoId()は動作確認済み
      let id = vm.autoId();
      this.cy.add([
        {
          group : 'nodes',
          data : {
            id : id, 
          },
          position : {
            x : x,
            y : y,
          },
          style : {
            'background-image': result.svg,
            'width': result.width,
            'height' : result.height,
          }
        }
      ]);
    },
    get_prop(speakers) {
      let result = {}
      for (let i = 0; i < speakers.length; i++) {
        if (Object.keys(result).indexOf(speakers[i].toString()) !== -1) {
          result[toString(speakers[i])] = 0;
        }
        else {
          result[toString(speakers[i])] += 1;
        }
      }
      return result;
    },
    makeSVG(speakers) {
      //circle
      const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
      circle.setAttribute('cx', 100);
      circle.setAttribute('cy', 100);
      circle.setAttribute('r', 80);
      circle.setAttribute('fill', 'none');
      circle.setAttribute('stroke', '#b22222');
      circle.setAttribute('stroke-width', 5);
      circle.setAttribute('stroke-dasharray', "none");//破線 10,10 etc
      circle.setAttribute('opacity', 1);
      circle.setAttribute('fill-opacity', 1);
      circle.setAttribute('stroke-opacity', 1);
      circle.setAttribute('transform', "rotate(0)");
      let width = 50;
      let height = 50;
      const area = document.getElementById('area');
      area.appendChild(circle);
      console.log(circle)
      return {circle, width, height}
    },
    set_data() {
      console.log('set_data')
      this.chart = !this.chart;
      this.chartData = {
        labels: ['January', 'February'],
        datasets: [
          {
            label: 'Data One',
            backgroundColor: ['#FF6384','#36A2EB'],
            borderColor: 'transparent' ,
            data: [Math.floor(Math.random() * (50 - 5 + 1)) + 5, 20]
          }
        ]
      }
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
        vm.set_data();
        vm.on_edge = true;
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
    });
    // window.addEventListener('click', (e) => {
    //   let flag = true;
    //   if(e.target.closest('dialog')) {
    //     console.log('on dialog');
    //   } 
    //   else {
    //     if (this.chart&&this.on_edge&&flag===true) {
    //       this.chart = false;
    //       console.log('not on dialog, but on edge');
    //     }
    //     else if (!this.chart&&this.on_edge&&flag===true) {
    //       //無視
    //       console.log('not on dialog, and not on edge');
    //       this.tap_edge();
    //     }
    //     else {
    //       this.chart = false;
    //       console.log('not on dialog');
    //     }
    //   }
    // })
  }
}
</script>
<style>
#cy {
  background-color: #f3f3f2;
  height: calc(100vw/3);
}

.dialog {
  z-index: 5;
  position: fixed;
  inset: 0;
  margin: auto;
  width: 400px;
  height: 400px;
}
</style>
