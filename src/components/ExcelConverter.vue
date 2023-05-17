<script>
import { defineComponent } from "vue";
import readXlsxFile, { Email, Integer, URL } from "read-excel-file";

export default defineComponent({
  name: "ExcelConverter",
  data: function () {
    return {
      file: null,
      activeTab: "1", // 0 = File, 1 = Columns, 2 = Data
      extractedHeaders: [],
      extractedData: [],
      columns: [
        {
          key: "Mi encabezado",
          prop: "foo",
          type: String,
          required: true,
        },
      ],
    };
  },
  methods: {
    /**
     * Change tabs
     */
    onTabChange(tab) {
      this.activeTab = tab;
    },
    /**
     * Updates the columns array with default values when the columnsCount changes
     */
    onColumnsCountChange(countValue) {
      const count = parseInt(countValue);
      console.log("onColumnsCountChange", count);
      this.columns = Array.from(Array(count).keys()).map((index) => {
        return {
          key: `Columna ${index + 1}`,
          prop: `column_${index + 1}`,
          type: "String",
          required: true,
        };
      });
    },
    /**
     * Receives a file type string and returns the corresponding type for readXlsxFile schema
     */
    getType(typeFromSelect) {
      switch (typeFromSelect) {
        case "String":
          return String;
        case "Number":
          return Number;
        case "Boolean":
          return Boolean;
        case "Date":
          return Date;
        case "Integer":
          return Integer;
        case "Email":
          return Email;
        case "URL":
          return URL;
        default:
          return String;
      }
    },
    /**
     * Reads the file and extracts the headers from the first row
     */
    extractHeaders(e) {
      this.file = e.target.files[0];
      readXlsxFile(e.target.files[0]).then((rows) => {
        this.extractedHeaders = rows[0];
        this.columns = this.extractedHeaders.map((header) => {
          return {
            key: header,
            prop: header.toLowerCase(),
            type: "String",
            required: true,
          };
        });
      });
    },
    /**
     * When the file changes, extract the headers from the first row to pre-populate the columns array
     */
    onFileChange(e) {
      this.extractHeaders(e);
    },
    /**
     * Reads the file and extracts the data from the rows
     */
    async extractData() {
      console.log("extractData", this.file);
      const res = await readXlsxFile(this.file, {
        schema: this.schema,
      });
      this.extractedData = res.rows;
    },
    logItem(item) {
      console.log(item);
    },
  },
  computed: {
    schema() {
      const schema = {};
      this.columns.forEach((column) => {
        schema[column.key] = {
          prop: column.prop,
          type: this.getType(column.type),
          required: column.required,
        };
      });
      return schema;
    },
    headers() {
      return this.extractedHeaders
        .map((header) => {
          return {
            text: header,
            value: this.schema[header].prop,
          };
        })
        .concat([
          {
            text: "",
            value: "actions",
            sortable: false,
          },
        ]);
    },
  },
});
</script>

<template>
  <v-responsive>
    <v-tabs fixed-tabs class="mb-6" v-model="activeTab">
      <v-tab>Archivo</v-tab>
      <v-tab>Columnas</v-tab>
      <v-tab>Datos</v-tab>
    </v-tabs>
    <v-row v-if="activeTab === 0" align="baseline">
      <v-col cols="12" md="6">
        <input type="file" id="fileInput" name="file" @change="onFileChange" />
      </v-col>
      <v-col cols="12" md="6">
        <v-text-field
          dense
          label="Especificar número de columnas"
          type="number"
          min="1"
          step="1"
          @change="onColumnsCountChange"
          hint="Cambiar el número de columnas aquí sobreescribe las detectadas automáticamente"
          persistent-hint
        ></v-text-field>
      </v-col>
    </v-row>
    <v-row
      v-for="(column, index) in columns"
      :key="index"
      v-show="activeTab === 1"
      class="mb-6"
    >
      <v-col cols="12">
        <p class="text-overline">Columna {{ index + 1 }}</p>
      </v-col>
      <v-col cols="6" md="3">
        <v-text-field
          dense
          label="Encabezado"
          v-model="column.key"
        ></v-text-field>
      </v-col>
      <v-col cols="6" md="3">
        <v-text-field dense label="`key`" v-model="column.prop"></v-text-field>
      </v-col>
      <v-col cols="6" md="3">
        <v-select
          dense
          label="Tipo"
          :items="[
            'String',
            'Number',
            'Boolean',
            'Date',
            'Integer',
            'Email',
            'URL',
          ]"
          v-model="column.type"
        ></v-select>
      </v-col>
      <v-col cols="6" md="3">
        <v-checkbox
          class="not-transparent"
          label="Requerido"
          v-model="column.required"
        ></v-checkbox>
      </v-col>
      <v-col cols="12" v-if="index !== columns.length - 1">
        <v-divider></v-divider>
      </v-col>
    </v-row>
    <v-row v-if="activeTab === 1">
      <v-spacer></v-spacer>
      <v-col cols="auto">
        <v-btn color="primary" @click="extractData">
          Confirmar y convertir
        </v-btn>
      </v-col>
    </v-row>
    <v-data-table
      v-if="activeTab === 2"
      :headers="headers"
      :items="extractedData"
    >
      <template v-slot:item.actions="{ item }">
        <v-btn small @click="logItem(item)">Click</v-btn>
      </template>
    </v-data-table>
  </v-responsive>
</template>

<style>
/* Checkbox is transparent for some reason */
.not-transparent input {
  opacity: 1 !important;
}
</style>
