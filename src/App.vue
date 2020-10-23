<template>
  <div id="app">
    <div>
      <input ref="filePicker" @change="FilePicked" type="file" accept="text/csv" />
    </div>
    <div>
      <textarea v-if="deckList" v-model="deckList" readonly style="width: 100%; height: 500px;" />
    </div>
  </div>
</template>

<script>
import Papa from "papaparse";

export default {
  name: "App",

  data() {
    return {
      deckList: undefined,
    };
  },
  methods: {
    async FilePicked(event) {
      const file = event.target.files[0];
      if (!file) return;
      const csv = await this.parseCSVFile(file);
      // Throw away header
      const collection = ["Count,Name,Edition,Foil"];
      const deck = [];
      for (const fields of csv.data) {
        // const name = this.fixName(fields["name"]);
        const name = fields["name"];
        const set = this.fixSet(fields["set_name"]);
        collection.push(fields["quantity"] + ",\"" + name + "\",\"" + set + "\"," + fields["extras"]);
        deck.push(fields["quantity"] + " " + name); // + " (" + fields["set_code"].toUpperCase() + ") " + fields["collector_number"]);
      }
      this.deckList = deck.join("\n");
      const filename = file.name.replace(/\.[^/.]+$/, "");
      this.downloadFile(filename + " (deckbox collection).csv", collection.join("\n"));
    },
    async parseCSVFile(file) {
      return new Promise((resolve, reject) => {
        Papa.parse(file, {
          header: true,
          complete: resolve,
          error: reject,
        });
      });
    },
    downloadFile(filename, data) {
      const url = window.URL.createObjectURL(new Blob([data]));
      const link = document.createElement("a");
      link.href = url;
      link.setAttribute("download", filename);
      document.body.appendChild(link);
      link.click();
    },
    // Some set names don't match between Helvault and Deckbox. Case helvault, return deckbox.
    fixSet(setName) {
      switch (setName) {
        case "Magic 2014":
          return "Magic 2014 Core Set";
        case "Magic 2015":
          return "Magic 2015 Core Set";
        case "Modern Masters 2015":
          return "Modern Masters 2015 Edition";
        case "Modern Masters 2017":
          return "Modern Masters 2017 Edition";
        case "M19 Gift Pack":
          return "M19 Gift Pack Promos";
        default:
          return setName;
      }
    },
    // Not reliable, some cards need // some don't
    fixName(name) {
      const divider = name.lastIndexOf(" //");
      if (divider > 0) {
        return name.substr(0, divider);
      } else {
        return name;
      }
    },
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
