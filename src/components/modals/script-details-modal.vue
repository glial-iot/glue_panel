<template>
  <div>
    <v-dialog v-on:keydown.esc="hide()" tabindex="0" v-model="visible" max-width="900px">
      <v-card>
        <v-card-title>
          <div class="title text-xs-center">{{$options.filters.type2string(type)}} "{{name}}"</div>
          <v-spacer></v-spacer>
          <v-btn color="primary" flat @click="$refs.rename_script.show(uuid, type, name)">Rename</v-btn>
          <v-btn color="primary" v-if="type !== 'DRIVER'" flat @click="$refs.change_object.show(uuid, object, type)">
            Change Object
          </v-btn>
          <v-btn color="primary" flat v-if="type === 'BUS_EVENT'" @click="run_script()">Run</v-btn>
        </v-card-title>
        <v-divider></v-divider>
        <v-card-text>
          <div class="row">
            <div class="subheading">UUID: {{uuid}}</div>
            <div class="subheading">Status: {{active_flag}}, {{status}}</div>
            <div class="subheading" v-if="type !== 'DRIVER'">{{$options.filters.object_label(type)}}: "{{object}}"</div>
            <div class="subheading">Message: "{{status_msg}}"</div>
            <div class="subheading">System load percent(relative): {{worktime_percent}}%</div>
            <div class="subheading">System load percent(absolute): {{alltime_percent}}%</div>
            <div class="subheading">
              Tag: {{tags}}<span class="grey--text" v-if="tags ===''">(Empty)</span>
              <span class="change_tags_button" @click="$refs.change_tag.show(uuid, tags, type)">
                        <i class="fa fa-edit blue--text"></i>
                     </span>
            </div>
            <div class="subheading">
              Comment: {{comment}}<span class="grey--text" v-if="comment ===''">(Empty)</span>
              <span class="change_comment_button" @click="$refs.change_comment.show(uuid, comment, type)">
                        <i class="fa fa-edit blue--text"></i>
                     </span>
            </div>
          </div>
        </v-card-text>

        <small-logs-table :uuid="uuid"></small-logs-table>

        <v-card-actions>
          <v-btn color="green darken-1" flat @click.stop="hide()">Close</v-btn>
        </v-card-actions>

      </v-card>
    </v-dialog>
    <rename-script-modal ref="rename_script" :hideDetails="hide" :updateName="update_name"></rename-script-modal>
    <change-object-modal ref="change_object" :updateObject="update_object"></change-object-modal>
    <change-tag-modal ref="change_tag" @tags_updated="get_script_data"></change-tag-modal>
    <change-comment-modal ref="change_comment" @comment_updated="get_script_data"></change-comment-modal>
  </div>
</template>

<script>
  import Vue from "vue";
  import Axios from "axios";
  import VueAxios from "vue-axios";
  import VueTimers from "vue-timers";

  Vue.use(VueAxios, Axios, VueTimers);

  import renameScriptModal from "./rename-script-modal.vue";
  import changeObjectModal from "./change-object-modal.vue";
  import changeTagModal from "./change-tag-modal.vue";
  import changeCommentModal from "./change-comment-modal.vue";
  import buttonInfo from "../buttons/button-info.vue";
  import smallLogsTable from "../small_logs_table.vue";

  export default {
    components: {
      renameScriptModal,
      changeObjectModal,
      changeTagModal,
      changeCommentModal,
      buttonInfo,
      smallLogsTable
    },
    data: () => ({
      visible: false,
      name: "",
      status: "",
      status_msg: "",
      worktime_percent: "",
      alltime_percent: "",
      type: "",
      uuid: "",
      active_flag: "",
      object: "",
      tags: "",
      comment: "",
      headers: [
        {
          text: "Info",
          value: "info",
          align: "center",
          sortable: false,
          width: "7%"
        },
        {
          text: "Level",
          value: "level",
          align: "center",
          sortable: false,
          width: "15%"
        },
        {
          text: "Date",
          value: "date",
          align: "center",
          sortable: false,
          width: "30%"
        },
        {
          text: "Entry",
          value: "entry",
          align: "left",
          sortable: false
        }
      ]
    }),
    timers: {
      get_script_data: {
        autostart: false,
        repeat: true,
        time: 1000
      }
    },
    methods: {
      show(item) {
        this.type = item.type;
        this.uuid = item.uuid;
        this.visible = true;
        this.get_script_data();
        this.$timer.start("get_script_data");
      },
      get_script_data() {
        Vue.axios
          .get(
            this.$store.getters.server_url +
            this.$store.state.endpoints[this.type],
            {
              params: {
                action: "get",
                uuid: this.uuid
              }
            }
          )
          .then(response => {
            this.active_flag = response.data.active_flag;
            this.name = response.data.name;
            this.object = response.data.object;
            this.type = response.data.type;
            this.status = response.data.status;
            this.status_msg = response.data.status_msg;
            this.active_flag = response.data.active_flag;
            this.tags = response.data.tag;
            this.comment = response.data.comment;
            this.object = response.data.object;
            this.worktime_percent = response.data.worktime_percent;
            this.alltime_percent = response.data.alltime_percent;
          })
          .catch(error => {
          });
      },
      run_script() {
        Vue.axios
          .get(
            this.$store.getters.server_url +
            this.$store.state.endpoints[this.type],
            {
              params: {
                action: "run_once",
                uuid: this.uuid
              }
            }
          )
          .then(response => {
            if (response.data.error_msg) {
              throw new Error(response.data.error_msg);
            }
          })
          .catch(error => {
            console.log(error);
          });
      },
      hide() {
        this.visible = false;
        this.name = "";
        this.status = "";
        this.status_msg = "";
        this.type = "";
        this.uuid = "";
        this.active_flag = "";
        this.object = "";
        this.tags = "";
        this.comment = "";
        this.worktime_percent = "";
        this.alltime_percent = "";
        this.$timer.stop("get_script_data");
      },
      update_name(value) {
        this.name = value;
      },
      update_object(value) {
        this.object = value;
      }
    },
    watch: {
      visible: function (val) {
        if (val === false) {
          this.$timer.stop("get_script_data");
        }
      }
    }
  };
</script>
