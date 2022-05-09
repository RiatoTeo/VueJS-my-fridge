<template>
  <v-app>
    <v-main>
      <v-app-bar>
        <v-app-bar-title>My fridge</v-app-bar-title>

        <v-spacer></v-spacer>

        <v-btn icon @click="this.searchButton = !this.searchButton">
          <v-icon>mdi-magnify</v-icon>
        </v-btn>

        <v-btn icon @click="showAddDialog()">
          <v-icon>mdi-plus</v-icon>
        </v-btn>
        <v-btn icon @click="showQRDialog()">
          <v-icon>mdi-barcode</v-icon>
        </v-btn>
      </v-app-bar>
      <v-dialog v-model="QRDialog.show">
        <div id="qr"></div>
      </v-dialog>

      <v-dialog v-model="addDialog.show" style="width: 100%; max-width: 400px">
        <v-card>
          <v-card-text>
            <v-form ref="addFormRef" v-model="addDialog.valid">
              <v-text-field
                class="text-h5 grey lighten-2"
                label="Name"
                type="text"
                :rules="addDialog.nameRules"
                v-model="addDialog.item.name"
              />

              <v-text-field
                class="text-h5 grey lighten-1"
                label="Quantity"
                type="number"
                :rules="addDialog.quantityRules"
                v-model="addDialog.item.quantity"
              />

              <v-text-field
                class="text-h5 grey lighten-1"
                label="giorni per scadenza"
                type="number"
                :rules="addDialog.scadenzaRules"
                v-model="addDialog.item.scadenzaGiorno"
              />

              <v-select
                @change="updateScadenza"
                :items="addDialog.scadenze"
                v-model="addDialog.item.scadenzaTemporale"
                label="tempo per scadenza"
              ></v-select>
            </v-form>

            today is:
            {{ addDialog.item.scadenzaCalcolata }}
          </v-card-text>

          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>

            <v-row align="center" justify="space-around">
              <v-btn
                color="success"
                @click="saveElement"
                v-if="addDialog.item.edit"
              >
                <v-icon left> mdi-content-save-edit </v-icon>
                Save
              </v-btn>

              <v-btn color="success" @click="saveElement" v-else>
                <v-icon left> mdi-plus-circle </v-icon>
                Add
              </v-btn>

              <v-btn color="error" @click="closeAddDialog">
                <v-icon left> mdi-close-circle </v-icon>
                Close
              </v-btn>
            </v-row>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <div id="container">
        <v-card>
          <v-card-title>
            <v-text-field
              v-model="searchSome"
              class="my-3"
              append-icon="mdi-magnify"
              label="Search"
              single-line
              hide-details
              v-if="this.searchButton"
            >
            </v-text-field>
          </v-card-title>
        </v-card>

        <v-list>
          <v-list-subheader>FRIGO</v-list-subheader>
          <v-list-item
            v-for="(item, i) in elements"
            :key="i"
            :value="item"
            active-color="primary"
          >
            <v-list-item-avatar start>
              <v-icon> mdi-food </v-icon>
            </v-list-item-avatar>
            <v-list-item-title v-text="item.name"></v-list-item-title>
            <v-list-item-title
              class="ml-5"
              v-text="item.quantity"
            ></v-list-item-title>
            <v-chip label :color="getColor(item.scadenzaTemporale)" class="ms-auto">
              <span class="mdi mdi-shield-alert me-1"></span>
              {{ item.scadenzaGiorno }}
              {{ item.scadenzaTemporale }}
            </v-chip>

            <v-btn
              color="error"
              dark
              icon
              @click="deleteElement(i)"
              class="mx-3"
            >
              <span class="mdi mdi-delete"></span>
            </v-btn>
            <v-btn fab dark large color="cyan" icon @click="editElement(i)">
              <span class="mdi mdi-pencil-outline"></span>
            </v-btn>
          </v-list-item>
        </v-list>
        <v-data-table
            :headers="addDialog.item.scadenzaTemporale"
            :items="elements"
            :search="searchSome"
          ></v-data-table>
      </div>
    </v-main>
  </v-app>
</template>

<script>
// import HelloWorld from "./components/HelloWorld.vue";
import { Html5QrcodeScanner } from "html5-qrcode";
import moment from "moment";

export default {
  name: "App",

  components: {
    // HelloWorld,
  },

  data: () => ({
    html5QrcodeScanner: null,
    searchButton: false,  //per barra ricerca
    searchSome: "", //string ctrl f

    // headers: [
    //       {
    //         text: 'Dessert (100g serving)',
    //         align: 'start',
    //         sortable: false,
    //         value: 'name',
    //       },
    //       { text: 'Calories', value: 'calories' },
    //       { text: 'Fat (g)', value: 'fat' },
    //       { text: 'Carbs (g)', value: 'carbs' },
    //       { text: 'Protein (g)', value: 'protein' },
    //       { text: 'Iron (%)', value: 'iron' },
    // ],

    QRDialog: {
      show: false,
    },
    addDialog: {
      show: false,
      valid: true,
      nameRules: [
        (value) => !!value || "Inserisci nome.",
        (value) => (value && value.length >= 3) || "Almeno 3 caratteri",
      ],
      quantityRules: [
        (value) => !!value || "Inserisci quantità.",
        (value) => (value && value >= 0) || "Inserisci valore valido",
      ],
      scadenzaRules: [
        (value) => !!value || "Inserisci quantità.",
        (value) =>
          (value && value > 0 && value <= 12) || "Inserisci valore valido",
      ],
      item: {
        name: "",
        quantity: 0,
        scadenzaGiorno: "",
        scadenzaTemporale: "",
        //barcode: "",
        edit: false,
        scadenzaCalcolata: moment(),
      },
      scadenze: ["set", "mes", "ann"],
    },
    elements: [],
  }),

  mounted() {
    console.log(moment().format());
    let elements = JSON.parse(localStorage.getItem("elements"));
    if (elements !== null) this.elements = elements;
  },

  methods: {
    closeAddDialog() {
      this.addDialog.show = false;
    },
    showAddDialog() {
      this.addDialog.show = true;
      this.addDialog.item = {
        name: "",
        quantity: "",
        scadenzaGiorno: "",
        scadenzaTemporale: "",
        scadenzaCalcolata: moment().format("DD/MM/YYYY"),
        edit: false,
      };
    },
    showQRDialog() {
      this.QRDialog.show = true;

      setTimeout(() => {
        this.html5QrcodeScanner = new Html5QrcodeScanner(
          "qr",
          { fps: 1, qrbox: { width: 250, height: 250 } },
          /* verbose= */ true
        );
        this.html5QrcodeScanner.render(this.onScanSuccess);
      }, 1000);
    },

    onScanSuccess(decodedText, decodedResult) {
      // handle the scanned code as you like, for example:
      console.log(`Code matched = ${decodedText}`, decodedResult);
      this.html5QrcodeScanner.stop();
    },

    updateScadenza(e) {
      console.log(e);

      this.addDialog.item.scadenzaCalcolata.add("1", "day");
    },

    saveElement() {
      if (
        this.addDialog.item.name === "" ||
        this.addDialog.item.quantity === "" ||
        this.addDialog.item.scadenzaGiorno === "" ||
        this.addDialog.item.scadenzaTemporale === ""
      ) {
        return;
      }

      if (this.addDialog.item.edit !== true) {
        this.elements.push(this.addDialog.item);
      }
      this.closeAddDialog();
      this.save();
    },

    deleteElement(index) {
      this.elements.splice(index, 1);
      this.save();
    },

    editElement(index) {
      // this.addDialog.item.scadenzaCalcolata = this.addDialog.item.scadenzaCalcolata.format("DD/MM/YYYY");
      this.showAddDialog();
      this.addDialog.item = this.elements[index];
      this.addDialog.item.edit = true;
      this.save();
    },

    getColor(scad) {
      if (scad === "set") return'red';
      else if(scad === "mes") return 'orange';
      else return 'green';
    },

    save() {
      localStorage.setItem("elements", JSON.stringify(this.elements));
    },
  },
};
</script>

<style lang="scss">
.v-overlay__content {
  width: 100%;
  max-width: 360px;
}

.v-card-title {
  align-items: center;
  display: flex;
  flex: none;
  font-size: 1.25rem;
  font-weight: 250;
  hyphens: auto;
  letter-spacing: 0.0125em;
  overflow-wrap: normal;
  padding: 0.5rem 1rem;
  text-transform: none;
  word-break: normal;
  word-wrap: break-word;
}
</style>