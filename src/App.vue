<template>
  <div id="app">
    <editor v-model="content" :merge-tags="mergeTags" ref="editor"/>
    <pre>{{content}}</pre>
    <div>Add a merge tag by clicking one of the following buttons:</div>
    <button
      v-for="(mergeTag, key) in mergeTags"
      :key="key"
      @click="$refs['editor'].addMergeTag(key, mergeTag)"
      v-text="mergeTag"
    />
    <br><br>

    Or type one of the following merge texts in the field above:<br>
    <span class="mergeTag" v-for="(mergeTag, index) in Object.keys(mergeTags)" :key="index" v-text="mergeTag" />

    <br><br>

    <small>This is a work in progress. Or even more a proof of concept. Don't use it in production. <br><br>
    MichaelTeeuw.nl - 2018</small>
  </div>
</template>

<script>
import Editor from './components/Editor.vue'

export default {
  name: 'app',
  components: {
    Editor
  },
  data () {
    return {
      content: 'The client is called $name, he works for $company and is $age years old.',
      mergeTags: {
        '$name': 'Client Name',
        '$company': 'Company',
        '$age': 'Age'
      }
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  padding: 0 100px;
}
.editor {
  margin-bottom: 20px;
}
.mergeTag {
  display: inline-block;
  margin: 10px;
  font-family: monospace;
  font-weight: bold;
  font-size: 1.2rem;

}
pre {
  text-align: left;
  padding: 5px;
  background-color: #eee;
  border-radius: 3px;
  // white-space: normal;
}
button {
  border: 1px solid #334;
  background-color: #445;
  margin: 3px;
  border-radius: 5px;
  color: white;
  min-width: 100px;
  font-size: 0.9rem;
  line-height: 1.5rem;

}
</style>
