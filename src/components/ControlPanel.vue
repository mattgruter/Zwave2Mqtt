/* eslint-disable */

<template>
  <v-container fluid>
    <v-card>
      <v-card-text>
        <v-container fluid>
          <v-layout>
            <v-flex xs12 sm3 md2 mr-2>
              <v-text-field label="Home ID" disabled v-model="homeid"></v-text-field>
            </v-flex>
            <v-flex xs12 sm3 md2>
              <v-text-field label="Home Hex" disabled v-model="homeHex"></v-text-field>
            </v-flex>
          </v-layout>

          <v-layout>
            <v-flex xs12 sm3 md2 mr-2>
              <v-text-field label="Controller status" disabled v-model="cnt_status"></v-text-field>
            </v-flex>

            <v-flex xs12 sm6 md4>
              <v-select
                label="Actions"
                append-outer-icon="send"
                v-model="cnt_action"
                :items="cnt_actions.concat(node_actions)"
                @click:append-outer="sendCntAction"
              ></v-select>
            </v-flex>

            <v-flex xs12 sm6 md3 align-self-center>
              <v-btn icon @click.native="importConfiguration">
                <v-tooltip bottom>
                  <v-icon dark color="primary" slot="activator">file_upload</v-icon>
                  <span>Import nodes.json Configuration</span>
                </v-tooltip>
              </v-btn>
              <v-btn icon @click.native="exportConfiguration">
                <v-tooltip bottom>
                  <v-icon dark color="primary" slot="activator">file_download</v-icon>
                  <span>Export nodes.json Configuration</span>
                </v-tooltip>
              </v-btn>
            </v-flex>
          </v-layout>
        </v-container>

        <v-layout row wrap>
          <v-flex xs12 ml-2>
            <v-switch label="Show hidden nodes" v-model="showHidden"></v-switch>
          </v-flex>
        </v-layout>

        <v-data-table
          :headers="headers"
          :items="tableNodes"
          :rows-per-page-items="[10, 20, {'text':'All','value':-1}]"
          item-key="node_id"
          class="elevation-1"
        >
          <template slot="items" slot-scope="props">
            <tr
              style="cursor:pointer;"
              :active="selectedNode == props.item"
              @click="selectedNode == props.item ? selectedNode = null : selectedNode = props.item"
            >
              <td>{{ props.item.node_id }}</td>
              <td>{{ props.item.type }}</td>
              <td>{{ props.item.ready ? (props.item.product + ' (' + props.item.manufacturer + ')') : '' }}</td>
              <td>{{ props.item.name || '' }}</td>
              <td>{{ props.item.loc || '' }}</td>
              <td>{{ props.item.status}}</td>
            </tr>
          </template>
        </v-data-table>

        <v-toolbar tabs style="margin-top:10px" class="elevation-1">
          <v-tabs v-model="currentTab" color="transparent" fixed-tabs>
            <v-tab key="node">Node</v-tab>
            <v-tab key="groups">Groups</v-tab>
            <v-tab key="scenes">Scenes</v-tab>
            <v-tab key="debug">Debug</v-tab>
          </v-tabs>
        </v-toolbar>

        <!-- TABS -->

        <v-tabs-items class="elevation-1" v-model="currentTab">
          <!-- TAB NODE INFO -->
          <v-tab-item key="node">
            <v-container v-if="selectedNode" fluid>
              <v-layout row>
                <v-flex xs3>
                  <v-select
                    label="Node actions"
                    append-outer-icon="send"
                    v-model="node_action"
                    :items="node_actions"
                    @click:append-outer="sendNodeAction"
                  ></v-select>
                </v-flex>
              </v-layout>

              <v-layout row>
                <v-flex xs2>
                  <v-subheader>Name: {{selectedNode.name}}</v-subheader>
                </v-flex>
                <v-flex xs4>
                  <v-text-field
                    label="New name"
                    append-outer-icon="send"
                    :error="!!nameError"
                    :error-messages="nameError"
                    v-model.trim="newName"
                    @click:append-outer="updateName"
                  ></v-text-field>
                </v-flex>
              </v-layout>

              <v-layout row>
                <v-flex xs2>
                  <v-subheader>Location: {{selectedNode.loc}}</v-subheader>
                </v-flex>
                <v-flex xs4>
                  <v-text-field
                    label="New Location"
                    append-outer-icon="send"
                    v-model.trim="newLoc"
                    :error="!!locError"
                    :error-messages="locError"
                    @click:append-outer="updateLoc"
                  ></v-text-field>
                </v-flex>
              </v-layout>

              <v-layout v-if="selectedNode.values" column>
                <v-subheader>Values</v-subheader>

                <!-- USER VALUES -->
                <v-expansion-panel class="elevation-0">
                  <v-expansion-panel-content>
                    <div slot="header">User</div>
                    <v-card>
                      <v-card-text>
                        <v-flex
                          v-for="(v, index) in selectedNode.values.filter(v => v.genre == 'user')"
                          :key="index"
                          xs12
                        >
                          <ValueID
                            @updateValue="updateValue"
                            v-model="selectedNode.values[selectedNode.values.indexOf(v)]"
                          ></ValueID>
                        </v-flex>
                      </v-card-text>
                    </v-card>
                  </v-expansion-panel-content>
                </v-expansion-panel>

                <v-divider></v-divider>

                <!-- CONFIG VALUES -->
                <v-expansion-panel class="elevation-0">
                  <v-expansion-panel-content>
                    <div slot="header">Configuration</div>
                    <v-card>
                      <v-card-text>
                        <v-flex
                          v-for="(v, index) in selectedNode.values.filter(v => v.genre == 'config')"
                          :key="index"
                          xs12
                        >
                          <ValueID
                            @updateValue="updateValue"
                            v-model="selectedNode.values[selectedNode.values.indexOf(v)]"
                          ></ValueID>
                        </v-flex>
                      </v-card-text>
                    </v-card>
                  </v-expansion-panel-content>
                </v-expansion-panel>

                <v-divider></v-divider>

                <!-- SYSTEM VALUES -->
                <v-expansion-panel class="elevation-0">
                  <v-expansion-panel-content>
                    <div slot="header">System</div>
                    <v-card>
                      <v-card-text>
                        <v-flex
                          v-for="(v, index) in selectedNode.values.filter(v => v.genre == 'system')"
                          :key="index"
                          xs12
                        >
                          <ValueID
                            @updateValue="updateValue"
                            v-model="selectedNode.values[selectedNode.values.indexOf(v)]"
                          ></ValueID>
                        </v-flex>
                      </v-card-text>
                    </v-card>
                  </v-expansion-panel-content>
                </v-expansion-panel>

                <v-divider></v-divider>
              </v-layout>

              <v-layout v-if="hassDevices.length > 0" column>
                <v-subheader>Home Assistant - Devices</v-subheader>

                <!-- HASS DEVICES -->
                <v-layout v-if="hassDevices.length > 0" raw wrap>
                  <v-flex xs12 md6 pa-1>
                    <v-btn color="blue darken-1" flat @click.native="storeDevices">Store</v-btn>

                    <v-data-table :headers="headers_hass" :items="hassDevices" class="elevation-1">
                      <template slot="items" slot-scope="props">
                        <tr
                          style="cursor:pointer;"
                          :active="selectedDevice == props.item"
                          @click="selectedDevice == props.item ? selectedDevice = null : selectedDevice = props.item"
                        >
                          <td class="text-xs">{{ props.item.id }}</td>
                          <td class="text-xs">{{ props.item.type }}</td>
                          <td class="text-xs">{{ props.item.object_id }}</td>
                          <td class="text-xs">{{ props.item.persistent ? 'Yes' : 'No' }}</td>
                        </tr>
                      </template>
                    </v-data-table>
                  </v-flex>
                  <v-flex xs12 md6 pa-1>
                    <v-btn v-if="!selectedDevice"
                      color="blue darken-1"
                      :disabled="errorDevice"
                      flat
                      @click.native="addDevice"
                    >Add</v-btn>
                    <v-btn v-if="selectedDevice"
                      color="blue darken-1"
                      :disabled="errorDevice"
                      flat
                      @click.native="updateDevice"
                    >Update</v-btn>
                    <v-btn v-if="selectedDevice"
                      color="green darken-1"
                      :disabled="errorDevice"
                      flat
                      @click.native="rediscoverDevice"
                    >Rediscover</v-btn>
                    <v-btn v-if="selectedDevice"
                      color="red darken-1"
                      :disabled="errorDevice"
                      flat
                      @click.native="deleteDevice"
                    >Delete</v-btn>
                    <v-textarea
                      label="Hass Device JSON"
                      auto-grow
                      :rules="[validJSONdevice]"
                      v-model="deviceJSON"
                    ></v-textarea>
                  </v-flex>
                </v-layout>
              </v-layout>
            </v-container>

            <v-container v-if="!selectedNode">
              <v-subheader>Click on a Node in the table</v-subheader>
            </v-container>
          </v-tab-item>

          <!-- TAB GROUPS -->
          <v-tab-item key="groups">
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12 sm6>
                  <v-select
                    label="Node"
                    v-model="group.node"
                    :items="nodes.filter(n => !n.failed)"
                    return-object
                    @change="resetGroup"
                    item-text="_name"
                  ></v-select>
                </v-flex>

                <v-flex v-if="group.node" xs12 sm6>
                  <v-select
                    label="Group"
                    v-model="group.group"
                    @input="getAssociations"
                    :items="group.node.groups"
                  ></v-select>
                </v-flex>

                <v-flex v-if="group.group" xs12 sm6>
                  <v-textarea
                    label="Current associations"
                    auto-grow
                    readonly
                    :value="group.associations"
                  ></v-textarea>
                </v-flex>

                <v-flex v-if="group.node" xs12 sm6>
                  <v-combobox
                    label="Target"
                    v-model="group.target"
                    :items="nodes.filter(n => !n.failed && n != group.node)"
                    return-object
                    hint="Select the node from the list or digit the node ID"
                    persistent-hint
                    item-text="_name"
                  ></v-combobox>
                </v-flex>

                <v-flex xs12 sm6>
                  <v-switch
                    label="Multi instance"
                    presistent-hint
                    hint="Enable this target node supports multi instance associations"
                    v-model="group.multiInstance"
                  ></v-switch>
                </v-flex>

                <v-flex v-if="group.multiInstance" xs12 sm6>
                  <v-text-field
                    v-model.number="group.targetInstance"
                    label="Instance ID"
                    hint="Target node instance ID"
                    type="number"
                  />
                </v-flex>

                <v-flex v-if="group.node && group.target && group.group" xs12>
                  <v-btn color="primary" @click.native="addAssociation" dark class="mb-2">Add</v-btn>
                  <v-btn color="primary" @click.native="removeAssociation" dark class="mb-2">Remove</v-btn>
                </v-flex>
              </v-layout>
            </v-container>
          </v-tab-item>

          <!-- TAB SCENES -->
          <v-tab-item key="scenes">
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12>
                  <v-btn flat @click.native="importScenes">
                    Import
                    <v-icon right dark color="primary">file_upload</v-icon>
                  </v-btn>
                  <v-btn flat @click.native="exportScenes">
                    Export
                    <v-icon right dark color="primary">file_download</v-icon>
                  </v-btn>
                </v-flex>

                <v-flex xs12 sm6>
                  <v-select
                    label="Scene"
                    v-model="selectedScene"
                    :items="scenesWithId"
                    item-text="label"
                    item-value="sceneid"
                  ></v-select>
                </v-flex>

                <v-flex xs12 sm6>
                  <v-text-field
                    label="New Scene"
                    append-outer-icon="send"
                    @click:append-outer="createScene"
                    v-model.trim="newScene"
                  ></v-text-field>
                </v-flex>

                <v-flex v-if="selectedScene" xs12>
                  <v-btn color="red darken-1" flat @click="removeScene">Delete</v-btn>
                  <v-btn color="green darken-1" flat @click="activateScene">Activate</v-btn>
                  <v-btn color="blue darken-1" flat @click="dialogValue = true">New Value</v-btn>
                </v-flex>
              </v-layout>

              <DialogSceneValue
                @save="saveValue"
                @close="closeDialog"
                v-model="dialogValue"
                :title="dialogTitle"
                :editedValue="editedValue"
                :nodes="nodes"
              />

              <v-data-table
                v-if="selectedScene"
                :headers="headers_scenes"
                :items="scene_values"
                class="elevation-1"
              >
                <template slot="items" slot-scope="props">
                  <td class="text-xs">{{ props.item.value_id }}</td>
                  <td class="text-xs">{{ props.item.node_id }}</td>
                  <td class="text-xs">{{ props.item.label }}</td>
                  <td class="text-xs">{{ props.item.value }}</td>
                  <td
                    class="text-xs"
                  >{{ props.item.timeout ? 'After ' + props.item.timeout + 's' : 'No' }}</td>
                  <td>
                    <v-icon small color="green" class="mr-2" @click="editItem(props.item)">edit</v-icon>
                    <v-icon small color="red" @click="deleteItem(props.item)">delete</v-icon>
                  </td>
                </template>
              </v-data-table>
            </v-container>
          </v-tab-item>

          <!-- TAB Debug -->
          <v-tab-item key="debug">
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12>
                  <v-btn color="green darken-1" flat @click="debugActive = true">Start</v-btn>
                  <v-btn color="red darken-1" flat @click="debugActive = false">Stop</v-btn>
                  <v-btn color="blue darken-1" flat @click="debug = []">Clear</v-btn>
                </v-flex>
                <v-flex xs12>
                  <div
                    id="debug_window"
                    style="height:400px;width:100%;overflow-y:scroll;"
                    class="body-1"
                    v-html="debug.join('')"
                  ></div>
                </v-flex>
              </v-layout>
            </v-container>
          </v-tab-item>
        </v-tabs-items>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
import { mapGetters, mapMutations } from "vuex";
import ConfigApis from "@/apis/ConfigApis";
import value from "@/apis/ConfigApis";

import ValueID from "@/components/ValueId";

import { default as AnsiUp } from "ansi_up";

const ansi_up = new AnsiUp();

import DialogSceneValue from "@/components/dialogs/DialogSceneValue";

//https://github.com/socketio/socket.io-client/blob/master/docs/API.md
import io from "socket.io-client";

const MAX_DEBUG_LINES = 300;

export default {
  name: "ControlPanel",
  components: {
    ValueID,
    DialogSceneValue
  },
  computed: {
    scenesWithId() {
      return this.scenes.map(s => {
        s.label = `[${s.sceneid}] ${s.label}` 
        return s
      })
    },
    dialogTitle() {
      return this.editedIndex === -1 ? "New Value" : "Edit Value";
    },
    tableNodes() {
      return this.showHidden ? this.nodes : this.nodes.filter(n => !n.failed);
    },
    hassDevices() {
      var devices = [];
      if (this.selectedNode && this.selectedNode.hassDevices) {
        for (const id in this.selectedNode.hassDevices) {
          var d = JSON.parse(JSON.stringify(this.selectedNode.hassDevices[id]));
          d.id = id;
          devices.push(d);
        }
      }

      return devices;
    }
  },
  watch: {
    dialogValue(val) {
      val || this.closeDialog();
    },
    newName(val) {
      this.nameError = /["+*\s]+/g.test(val)
        ? 'Remove " + * and blank space charaters'
        : null;
    },
    newLoc(val) {
      this.locError = /["+*\s]+/g.test(val)
        ? 'Remove " + * and blank space charaters'
        : null;
    },
    selectedNode() {
      if (this.selectedNode) {
        this.newName = this.selectedNode.name;
        this.newLoc = this.selectedNode.loc;
        this.node_action = null;
        this.selectedDevice = null;
      }
    },
    selectedDevice() {
      this.deviceJSON = this.selectedDevice
        ? JSON.stringify(this.selectedDevice, null, 2)
        : "";
    },
    selectedScene() {
      this.refreshValues();
    },
    currentTab() {
      if (this.currentTab == 2) {
        this.refreshScenes();
      } else {
        this.selectedScene = null;
        this.scene_values = [];
      }

      if (this.currentTab == 3) {
        this.debugActive = true;
      } else {
        this.debugActive = false;
      }
    }
  },
  data() {
    return {
      socket: null,
      nodes: [],
      scenes: [],
      debug: [],
      homeid: "",
      homeHex: "",
      showHidden: false,
      debugActive: false,
      selectedScene: null,
      cnt_status: "Unknown",
      newScene: "",
      scene_values: [],
      dialogValue: false,
      editedValue: {},
      editedIndex: -1,
      headers_scenes: [
        { text: "Value ID", value: "value_id" },
        { text: "Node", value: "node_id" },
        { text: "Label", value: "label" },
        { text: "Value", value: "value" },
        { text: "Timeout", value: "timeout" },
        { text: "Actions", sortable: false }
      ],
      headers_hass: [
        { text: "Id", value: "id" },
        { text: "Type", value: "type" },
        { text: "Object id", value: "object_id" },
        { text: "Persistent", value: "persistent" }
      ],
      selectedDevice: null,
      errorDevice: false,
      deviceJSON: "",
      group: {},
      currentTab: 0,
      node_action: "requestNetworkUpdate",
      node_actions: [
        {
          text: "Update neighbors",
          value: "requestNodeNeighborUpdate"
        },
        {
          text: "Get Node neighbors",
          value: "getNodeNeighbors"
        },
        {
          text: "Update return route",
          value: "assignReturnRoute"
        },
        {
          text: "Delete return routes",
          value: "deleteAllReturnRoutes"
        },
        {
          text: "Send NIF",
          value: "sendNodeInformation"
        },
        {
          text: "Request network update",
          value: "requestNetworkUpdate"
        },
        {
          text: "Node statistic",
          value: "getNodeStatistics"
        },
        {
          text: "Has node failed",
          value: "hasNodeFailed"
        },
        {
          text: "Replace failed node",
          value: "replaceFailedNode"
        },
        {
          text: "Heal node",
          value: "healNetworkNode"
        },
        {
          text: "Replication send",
          value: "replicationSend"
        },
        {
          text: "Test node",
          value: "testNetworkNode"
        }
      ],
      cnt_action: "healNetwork",
      cnt_actions: [
        {
          text: "Add Node (inclusion)",
          value: "addNode"
        },
        {
          text: "Remove Node (exclusion)",
          value: "removeNode"
        },
        {
          text: "Transfer primary role",
          value: "transferPrimaryRole"
        },
        {
          text: "Create new primary",
          value: "createNewPrimary"
        },
        {
          text: "Receive configuration",
          value: "receiveConfiguration"
        },
        {
          text: "Cancel Command",
          value: "cancelControllerCommand"
        },
        {
          text: "Heal Network",
          value: "healNetwork"
        },
        {
          text: "Driver statistic",
          value: "getDriverStatistics"
        },
        {
          text: "Hard reset",
          value: "hardReset"
        },
        {
          text: "Soft reset",
          value: "softReset"
        },
        {
          text: "Test network",
          value: "testNetwork"
        }
      ],
      newName: "",
      nameError: null,
      locError: null,
      newLoc: "",
      selectedNode: null,
      headers: [
        { text: "ID", value: "node_id" },
        { text: "Type", value: "type" },
        { text: "Product", value: "product" },
        { text: "Name", value: "name" },
        { text: "Location", value: "loc" },
        { text: "Status", value: "status" }
      ],
      rules: {
        required: value => {
          var valid = false;

          if (value instanceof Array) valid = value.length > 0;
          else valid = !isNaN(value) || !!value; //isNaN is for 0 as valid value

          return valid || "This field is required.";
        }
      }
    };
  },
  methods: {
    showSnackbar(text) {
      this.$emit("showSnackbar", text);
    },
    validJSONdevice() {
      var valid = true
      try {
        JSON.parse(this.deviceJSON);
      } catch (error) {
        valid = false;
      }
      this.errorDevice = !valid;

      return valid || "JSON test failed";
    },
    importConfiguration() {
      var self = this;
      if (
        confirm(
          "Attention: This will override all existing nodes names and location"
        )
      ) {
        self.$emit("import", "json", function(err, data) {
          if (!err && data) {
            ConfigApis.importConfig({ data: data })
              .then(data => {
                self.showSnackbar(data.message);
              })
              .catch(error => {
                console.log(error);
              });
          }
        });
      }
    },
    exportConfiguration() {
      var self = this;
      ConfigApis.exportConfig()
        .then(data => {
          self.showSnackbar(data.message);
          if (data.success) {
            self.$emit("export", data.data, "nodes", "json");
          }
        })
        .catch(error => {
          console.log(error);
        });
    },
    importScenes() {
      var self = this;
      if (
        confirm(
          "Attention! This operation will override all current scenes and cannot be undone"
        )
      ) {
        this.$emit("import", "json", function(err, scenes) {
          //TODO: add checks on file entries
          if (scenes instanceof Array) {
            self.apiRequest("setScenes", [scenes]);
          } else {
            self.showSnackbar("Imported file not valid");
          }
        });
      }
    },
    exportScenes() {
      this.$emit("export", this.scenes, "scenes");
    },
    apiRequest(apiName, args) {
      if (this.socket.connected) {
        var data = {
          api: apiName,
          args: args
        };
        this.socket.emit("ZWAVE_API", data);
      } else {
        this.showSnackbar("Socket disconnected");
      }
    },
    refreshValues() {
      if (this.selectedScene) {
        this.apiRequest("sceneGetValues", [this.selectedScene]);
      }
    },
    refreshScenes() {
      this.apiRequest("getScenes", []);
    },
    createScene() {
      if (this.newScene) {
        this.apiRequest("createScene", [this.newScene]);
        this.refreshScenes();
        this.newScene = "";
      }
    },
    removeScene() {
      if (this.selectedScene) {
        this.apiRequest("removeScene", [this.selectedScene]);
        this.selectedScene = null;
        this.refreshScenes();
      }
    },
    activateScene() {
      if (this.selectedScene) {
        this.apiRequest("activateScene", [this.selectedScene]);
      }
    },
    editItem(item) {
      this.editedIndex = this.scene_values.indexOf(item);
      var node = this.nodes[item.node_id];
      var value = node.values.find(v => v.value_id == item.value_id);

      value = Object.assign({}, value);
      value.newValue = item.value;

      this.editedValue = {
        node: node,
        value: value,
        timeout: this.scene_values[this.editedIndex].timeout
      };
      this.dialogValue = true;
    },
    deleteItem(value) {
      if (confirm("Are you sure you want to delete this item?")) {
        this.apiRequest("removeSceneValue", [
          this.selectedScene,
          value.node_id,
          value.class_id,
          value.instance,
          value.index
        ]);
        this.refreshValues();
      }
    },
    deleteDevice() {
      var device = this.selectedDevice;
      if (device && confirm("Are you sure you want to delete selected device?"))
        this.socket.emit("HASS_API", {
          apiName: "delete",
          device: device,
          node_id: this.selectedNode.node_id
        });
    },
    rediscoverDevice() {
      var device = this.selectedDevice;
      if (
        device &&
        confirm("Are you sure you want to re-discover selected device?")
      )
        this.socket.emit("HASS_API", {
          apiName: "discover",
          device: device,
          node_id: this.selectedNode.node_id
        });
    },
    updateDevice() {
      if (!this.errorDevice) {
        var updated = JSON.parse(this.deviceJSON);
        this.$set(
          this.selectedNode.hassDevices,
          this.selectedDevice.id,
          updated
        );
        this.socket.emit("HASS_API", {
          apiName: "update",
          device: updated,
          node_id: this.selectedNode.node_id
        });
      }
    },
    addDevice() {
      if (!this.errorDevice) {
        var newDevice = JSON.parse(this.deviceJSON)
        this.socket.emit("HASS_API", {
          apiName: "add",
          device: newDevice,
          node_id: this.selectedNode.node_id
        });
      }
    },
    storeDevices() {
      this.socket.emit("HASS_API", {
        apiName: "store",
        devices: this.selectedNode.hassDevices,
        node_id: this.selectedNode.node_id
      });
    },
    closeDialog() {
      this.dialogValue = false;
      setTimeout(() => {
        this.editedValue = {};
        this.editedIndex = -1;
      }, 300);
    },
    saveValue() {
      var value = this.editedValue.value;
      value.value = value.newValue;

      // if value already exists it will be updated
      this.apiRequest("addSceneValue", [
        this.selectedScene,
        value,
        value.value,
        this.editedValue.timeout
      ]);
      this.refreshValues();

      this.closeDialog();
    },
    sendCntAction() {
      if (this.cnt_action) {
        var args = [];
        var askId = this.node_actions.find(a => a.value == this.cnt_action);
        if (askId) {
          var id = parseInt(prompt("Node ID"));

          if (isNaN(id)) {
            this.showMessage("Node ID must be an integer value");
            return;
          }
          args.push(id);
        }

        if (this.cnt_action == "addNode") {
          var secure = confirm("Start inclusion in security mode?");
          args.push(secure);
        } else if(this.cnt_action == "hardReset") {
            var ok = confirm("Your controller will be reset to factory and all paired devices will be removed");
            if(!ok) {
                return;
            }
        }

        this.apiRequest(this.cnt_action, args);
      }
    },
    sendNodeAction() {
      if (this.selectedNode) {
        this.apiRequest(this.node_action, [this.selectedNode.node_id]);
      }
    },
    saveConfiguration() {
      this.apiRequest("writeConfig", []);
    },
    updateName() {
      if (this.selectedNode && !this.nameError) {
        this.apiRequest("setNodeName", [
          this.selectedNode.node_id,
          this.newName
        ]);
      }
    },
    updateLoc() {
      if (this.selectedNode && !this.locError) {
        this.apiRequest("setNodeLocation", [
          this.selectedNode.node_id,
          this.newLoc
        ]);
      }
    },
    resetGroup() {
      this.$set(this.group, "associations", []);
      this.$set(this.group, "group", -1);
    },
    getAssociations() {
      var g = this.group;
      if (g && g.node) {
        this.apiRequest("getAssociations", [g.node.node_id, g.group]);
      }
    },
    addAssociation() {
      var g = this.group;
      var target = !isNaN(g.target) ? parseInt(g.target) : g.target.node_id;

      if (g && g.node && target) {
        var args = [g.node.node_id, g.group, target];

        if (g.multiInstance) {
          args.push(g.targetInstance || 0);
        }

        this.apiRequest("addAssociation", args);

        // wait a moment before refresh to check if the node
        // has been added to the group correctly
        setTimeout(this.getAssociations, 500);
      }
    },
    removeAssociation() {
      var g = this.group;
      var target = !isNaN(g.target) ? parseInt(g.target) : g.target.node_id;
      if (g && g.node && target) {
        this.apiRequest("removeAssociation", [g.node.node_id, g.group, target]);
        // wait a moment before refresh to check if the node
        // has been added to the group correctly
        setTimeout(this.getAssociations, 500);
      }
    },
    updateValue(v) {
      this.apiRequest("setValue", [
        v.node_id,
        v.class_id,
        v.instance,
        v.index,
        v.type == "button" ? true : v.newValue
      ]);
      v.toUpdate = true;
    },
    jsonToList(obj) {
      var s = "";
      for (var k in obj) s += k + ": " + obj[k] + "\n";

      return s;
    },
    initNode(n) {
      var values = [];
      for (var k in n.values) {
        n.values[k].newValue = n.values[k].value;
        values.push(n.values[k]);
      }
      n.values = values;
      this.setName(n);
    },
    setName(n) {
      n._name = n.name || "NodeID_" + n.node_id;
    }
  },
  mounted() {
    var self = this;

    this.socket = io(ConfigApis.getSocketIP());

    this.socket.on("connect", () => {
      console.log("Socket connected");
      self.$emit("updateStatus", "Connected", "green");
    });

    this.socket.on("disconnect", () => {
      console.log("Socket closed");
      self.$emit("updateStatus", "Disconnected", "red");
    });

    this.socket.on("error", () => {
      console.log("Socket error");
    });

    this.socket.on("reconnecting", () => {
      console.log("Socket reconnecting");
      self.$emit("updateStatus", "Reconnecting", "yellow");
    });

    this.socket.on("CONTROLLER_CMD", data => {
      self.cnt_status = data.help;
    });

    this.socket.on("DRIVER_READY", info => {
      self.homeid = info.homeid;
      self.homeHex = info.name;
    });

    this.socket.on("NODE_REMOVED", node => {
      self.$set(self.nodes, node.node_id, node);
    });

    this.socket.on("DEBUG", data => {
      if (self.debugActive) {
        data = ansi_up.ansi_to_html(data);
        data = data.replace(/\n/g, "</br>");
        //\b[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}.[0-9]{3}Z\b
        self.debug.push(data);

        if (self.debug.length > MAX_DEBUG_LINES) self.debug.shift();

        var textarea = document.getElementById("debug_window");
        textarea.scrollTop = textarea.scrollHeight;
      }
    });

    this.socket.on("INIT", data => {
      //convert node values in array
      var nodes = data.nodes;
      for (var i = 0; i < nodes.length; i++) {
        self.initNode(nodes[i]);
      }
      self.nodes = nodes;
      self.cnt_status = data.error ? data.error : data.cntStatus;
      self.homeid = data.info.homeid;
      self.homeHex = data.info.name;
    });

    this.socket.on("NODE_UPDATED", data => {
      self.initNode(data);
      if (!self.nodes[data.node_id] || self.nodes[data.node_id].failed) {
        //add missing nodes
        while (self.nodes.length < data.node_id)
          self.nodes.push({
            node_id: self.nodes.length,
            failed: true,
            status: "Removed"
          });
      }
      self.$set(self.nodes, data.node_id, data);

      if (this.selectedNode && this.selectedNode.node_id === data.node_id)
        this.selectedNode = self.nodes[data.node_id];
    });

    this.socket.on("VALUE_UPDATED", data => {
      var node = self.nodes[data.node_id];
      if (node && node.values) {
        var index = node.values.findIndex(v => v.value_id == data.value_id);
        if (index >= 0) {
          if (self.nodes[data.node_id].values[index].toUpdate) {
            self.nodes[data.node_id].values[index].toUpdate = false;
            self.showSnackbar("Value updated");
          }

          if (!data.newValue) data.newValue = data.value;

          self.$set(self.nodes[data.node_id].values, index, data);
        }
      }
    });

    this.socket.on("API_RETURN", data => {
      if (data.success) {
        switch (data.api) {
          case "getAssociations":
            data.result = data.result.map(a => self.nodes[a]._name || a);
            self.$set(self.group, "associations", data.result.join("\n"));
            break;
          case "getScenes":
            self.scenes = data.result;
            break;
          case "setScenes":
            self.scenes = data.result;
            self.showSnackbar("Successfully updated scenes");
            break;
          case "sceneGetValues":
            self.scene_values = data.result;
            break;
          case "getNodeNeighbors":
            confirm("Node neighbors \n" + self.jsonToList(data.result));
            break;
          case "getDriverStatistics":
            confirm("Driver statistics \n" + self.jsonToList(data.result));
            break;
          case "getNodeStatistics":
            confirm("Node statistics \n" + self.jsonToList(data.result));
            break;
          default:
            self.showSnackbar("Successfully call api " + data.api);
        }
      } else {
        self.showSnackbar(
          "Error while calling api " + data.api + ": " + data.message
        );
      }
    });
  },
  beforeDestroy() {
    if (this.socket) this.socket.close();
  }
};
</script>
