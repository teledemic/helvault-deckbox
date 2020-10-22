<template>
  <div id="app">
    <input ref="filePicker" @change="FilePicked" type="file" accept="text/csv" />
  </div>
</template>

<script>
export default {
  name: "App",
  methods: {
    async FilePicked(event) {
      const file = event.target.files[0];
      if (!file) return;
      const csv = await this.readFileAsync(file);
      const lines = csv.split("\n");
      if (lines[0] !== "collector_number,extras,language,name,oracle_id,quantity,scryfall_id,set_code,set_name") {
        throw new Error("Invalid header, not a helvault file");
      }
      // Throw away header
      lines.shift();
      console.log(lines);
    },
    async readFileAsync(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = (e) => {
          resolve(e.target.result);
        }
        reader.onerror = reject;
        reader.readAsText(file);
      });
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
