<template>
  <div id="app">
    <div class="editor-container">
      <div class="components-area">
        <h5>Components</h5>
        <draggable
          v-model="allComponents"
          class="placeholder-components"
          :options="dragOptionsInitial"
        >
          <component
            class="draggable-component"
            v-for="(comp, index) in allComponents"
            :key="index"
            :is="comp.component"
            v-bind="comp.props"
          >
            <span v-html="comp.content"></span>
          </component>
        </draggable>
      </div>

      <div class="render-area">
        <h5>Render Area</h5>
        <draggable
          v-model="componentsToRender"
          class="rendered-components"
          :options="dragOptions"
        >
          <component-renderer
            class="draggable-component"
            v-for="(comp, index) in componentTree"
            :component-data="comp"
            :drag-options="dragOptions"
            :index="index"
            :key="index"
            @component-click="componentClick"
          >
          </component-renderer>
        </draggable>
      </div>
      <div class="edit-area">
        <component-editor
          :componentProps="selectedComponent.props"
          @change="syncProps"
        >
        </component-editor>
      </div>
    </div>

    <div class="json-container">
      <h5>Edit</h5>
      {{ JSON.stringify(componentTree, null, 2) }}
    </div>
  </div>
</template>

<script>
import Draggable from 'vuedraggable';
import camelCase from 'lodash/camelCase';
import deepClone from 'lodash/cloneDeep';
import { extractProps, extractSlots } from '../componentUtils';
import ComponentEditor from './components/ComponentEditor';
import ComponentRenderer from './components/ComponentRenderer';

export default {
  name: 'App',
  components: {
    Draggable,
    ComponentEditor,
    ComponentRenderer,
  },
  data() {
    return {
      selectedComponentIndex: 0,
      selectedComponent: {
        props: {},
        slots: [],
      },
      componentTree: [],
      allComponents: [
        {
          component: 'el-button',
          props: {
            type: 'primary',
          },
          content: 'Button',
          children: [],
        },
        {
          component: 'el-switch',
          children: [],
        },
        {
          component: 'el-card',
          content: 'card',
          children: [],
        },
        {
          component: 'el-input',
          props: {
            value: 'Input Value',
          },
          children: [],
        },
        {
          component: 'el-row',
          props: {
            class: 'row',
          },
          content: 'row',
          children: [],
        },
        {
          component: 'el-col',
          props: {
            class: 'col',
          },
          content: 'col',
          children: [],
        },
        {
          component: 'el-form',
          props: {
            class: 'demo-form',
          },
          children: [
            {
              component: 'el-form-item',
              props: {
                label: 'Input label',
              },
              children: [
                {
                  component: 'el-input',
                  props: {
                    value: 'Test',
                  },
                  children: [],
                },
              ],
            },
          ],
        },
      ],
      componentsToRender: [],
    };
  },
  computed: {
    dragOptionsInitial() {
      return {
        draggable: '.draggable-component',
        group: { name: 'components', pull: 'clone', put: false },
        ghostClass: 'draggable-component--ghost',
        chosenClass: 'draggable-component--chosen',
      };
    },
    dragOptions() {
      return {
        draggable: '.draggable-component',
        group: 'components',
        ghostClass: 'draggable-component--ghost',
        chosenClass: 'draggable-component--chosen',
      };
    },
  },
  watch: {
    componentsToRender: {
      deep: true,
      handler(newValue) {
        this.setComponentData(newValue);
      },
    },
  },
  mounted() {
    this.allComponents = this.allComponents.map(this.parseComponent);
  },
  methods: {
    setComponentData(value) {
      const tree = deepClone(value);
      this.componentTree = tree;
    },
    parseComponent(component) {
      const children = component.children.map(this.parseComponent);
      return {
        component: component.component,
        props: component.props,
        content: component.content,
        children,
        slots: this.getComponentSlots(component.component),
      };
    },
    syncProps(newValue) {
      this.componentTree[this.selectedComponentIndex].props = newValue;
    },
    updateContent(value) {
      // eslint-disable-next-line no-console
      console.log('Update', value);
    },
    getComponentSlots(compName) {
      const component = this.getComponent(compName);
      return extractSlots(component);
    },
    componentClick(compName, index) {
      const component = this.getComponent(compName);
      const props = extractProps(component);
      this.selectedComponent.props = props;
      this.selectedComponentIndex = index;
    },
    getComponent(compName) {
      let camelCaseComponentName = camelCase(compName);
      camelCaseComponentName =
        camelCaseComponentName.charAt(0).toUpperCase() +
        camelCaseComponentName.slice(1);
      const component = this.$root.$options.components[camelCaseComponentName];
      return component;
    },
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.editor-container {
  display: flex;
}
.render-area,
.components-area,
.edit-area {
  flex: 1;
  border: 1px solid #ebeef5;
  min-height: 70vh;
}

.json-container {
  min-height: 30vh;
}

.placeholder-components {
  display: flex;
  flex-direction: column;
}

.rendered-components {
  height: 100%;
}
.row {
  min-height: 50px;
  width: 100%;
  border: 1px solid #ebeef5;
}

.col {
  min-height: 50px;
  min-width: 50px;
  border: 1px solid #ebeef5;
}
.demo-form {
  min-height: 100px;
}
</style>
