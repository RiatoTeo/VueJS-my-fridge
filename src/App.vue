<template>
  <v-app>
    <v-main>
      <v-app-bar>
        <v-app-bar-title>My fridge</v-app-bar-title>

        <v-spacer></v-spacer>

        <v-btn icon>
          <v-icon>mdi-magnify</v-icon>
        </v-btn>

        <button @click="showNewEl()">
          <v-btn icon @click="addDialog = true">
            <v-icon>mdi-plus</v-icon>
          </v-btn>
        </button>
        <!--
        <v-btn icon>
          <v-icon>mdi-barcode</v-icon>
        </v-btn> -->
      </v-app-bar>

      <v-dialog v-model="addDialog">
        <v-card>
          <input class="text-h5 grey lighten-2" text v-model="items.name" />

          <v-divider></v-divider>

          <input class="text-h5 grey lighten-1" text v-model="items.quantity" />

          <v-divider></v-divider>

          <v-card-text>
            <textarea cols="30" rows="10" v-model="items.scadenza"></textarea>
          </v-card-text>

          <v-divider></v-divider>

          <!--<input class="text-h5 grey lighten-2" text v-model="items.barcode" /> -->

          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" text @click="addNewElement()"> Add </v-btn>
          </v-card-actions>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" text @click="addDialog = false">
              Close
            </v-btn>
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

            <button @click="deleteEl(i)" class="me-1">
              <span class="mdi mdi-delete"></span>
            </button>
            <button @click="editEl(i)">
              <span class="mdi mdi-pencil"></span>
            </button>
          </v-list-item>
        </v-list>
      </div>
    </v-main>
  </v-app>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";

export default {
  name: "App",

  components: {
    HelloWorld,
  },

  data: () => ({
    addDialog: false,
    items: {
      name: "",
      quantity: 0,
      scadenza: "",
      //barcode: "",
      edit: false,
    },

    elements: [],
  }),

  mounted() {
    let elements = JSON.parse(localStorage.getItem("elements"));
    if (elements !== null) this.elements = elements;
  },

  methods: {
    showNewEl() {
      this.addDialog = true;
      this.items = {
        name: "",
        quantity: "",
        scadenza: "",
        //barcode: "",
        edit: false,
      };
    },

    addNewElement() {
      if (
        this.items.quantity === "" ||
        this.items.name === "" ||
        this.items.scadenza === "" ||
        this.items.barcode === ""
      ) {
        this.items.edit = true;
      }
      if (this.items.edit !== true) {
        if (this.items.quantity !== "") {
          this.elements.push(this.items);
        }
      }
      this.addDialog = false;
      this.items = [
        {
          name: "",
          quantity: 0,
          scadenza: "",
          barcode: "",
        },
      ];
      this.save();
    },

    deleteEl(index) {
      this.elements.splice(index, 1);
      this.save();
    },

    editEl(index) {
      this.addDialog = true;
      this.items = this.elements[index];
      this.items.edit = true;
      this.save();
    },

    save() {
      localStorage.setItem("elements", JSON.stringify(this.elements));
    },
  },
};
</script>

<style>
</style>