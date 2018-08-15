<template>
   <div>
      <v-btn right color="secondary" class="my-2" @click="dialog_visible = true">
         <v-icon left small>fa-file</v-icon> Create
      </v-btn>

      <v-dialog v-model="dialog_visible" max-width="500px">
         <v-card>
            <v-card-title>
               <div class="title text-xs-center">Create new script</div>
            </v-card-title>
            <v-divider></v-divider>
            <v-card-text>
               <v-text-field autofocus label="Name" v-model="name"></v-text-field>
            </v-card-text>
            <v-card-text v-if="get_object_label(type)">
               <v-text-field :label="get_object_label(type)" v-model="object"></v-text-field>
               <span>Endpoint:
                  <a :href="$store.getters.server_url+'/we/'+object">{{$store.getters.server_url}}/we/{{object || "..."}}</a>
               </span>
            </v-card-text>
            <v-divider></v-divider>
            <v-card-actions>
               <v-btn color="error" flat @click.stop="close">Close</v-btn>
               <v-spacer></v-spacer>
               <v-btn color="primary" right flat @click="create_file()">
                  <v-icon left small>fa-file</v-icon> Create </v-btn>
            </v-card-actions>
         </v-card>
      </v-dialog>
      <snackbar ref="snackbar"></snackbar>
   </div>
</template>

<script>
import Vue from "vue";
import Axios from "axios";
import VueAxios from "vue-axios";
import { object_label } from "../../utils/index.js";
import snackbar from "../snackbar.vue";
Vue.use(VueAxios, Axios);

export default {
  components: {
    snackbar
  },
  data() {
    return {
      dialog_visible: false,
      name: "",
      object: ""
    };
  },
  props: ["type"],
  methods: {
    close() {
      this.name = "";
      this.dialog_visible = false;
    },
    create_file() {
      let params = {};

      if (this.type !== "DRIVER" && this.object === "") {
        this.$refs.snackbar.update(
          `Field "${this.get_object_label(this.type)}" is required`
        );
        return;
      }

      if (this.type === "DRIVER") {
        params = {
          action: "create",
          name: this.name,
          type: this.type
        };
      } else {
        params = {
          action: "create",
          name: this.name,
          type: this.type,
          object: this.object
        };
      }

      Vue.axios
        .get(this.$store.getters.server_url + "/scripts", { params })
        .then(response => {
          this.$emit("data_updated", this.item);
          this.dialog_visible = false;
          this.name = "";
        })
        .catch(error => {
          console.log(error);
          this.$emit("create_error");
        });
    },
    get_object_label(type) {
      return object_label(type);
    }
  }
};
</script>