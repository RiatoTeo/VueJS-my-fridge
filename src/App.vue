<template>
  <v-app>
    <v-main>
      <v-app-bar>
        <v-app-bar-title>My fridge</v-app-bar-title>

        <v-spacer></v-spacer>

        <v-btn icon>
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

              <v-select
                @change="updateScadenza"
                :items="addDialog.scadenze"
                v-model="addDialog.item.scadenza"
                label="Scadenza"
              ></v-select>
            </v-form>
            {{ addDialog.item.scadenzaCalcolata.format("DD/MM/YYYY") }}
          </v-card-text>

          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="primary"
              @click="saveElement"
              v-if="addDialog.item.edit"
            >
              Save
            </v-btn>
            <v-btn color="primary" @click="saveElement" v-else> Add </v-btn>

            <v-btn color="primary" @click="closeAddDialog"> Close </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <div id="container">
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
            <v-chip class="ms-auto">
              {{ item.scadenza }}
            </v-chip>
            <!--<v-list-item-title v-text="item.barcode"></v-list-item-title>-->

            <v-btn icon @click="deleteElement(i)" class="mx-1">
              <span class="mdi mdi-delete"></span>
            </v-btn>
            <v-btn icon @click="editElement(i)">
              <span class="mdi mdi-pencil"></span>
            </v-btn>
          </v-list-item>
        </v-list>
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
      item: {
        name: "",
        quantity: 0,
        scadenza: "",
        //barcode: "",
        edit: false,
        scadenzaCalcolata: moment(),
      },
      scadenze: ["1 settimana", "1 mese", "2 mesi", "3 mesi"],
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
        scadenza: "",
        scadenzaCalcolata: moment(),
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
      this.showAddDialog();
      this.addDialog.item = this.elements[index];
      this.addDialog.item.edit = true;
      this.save();
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
</style>