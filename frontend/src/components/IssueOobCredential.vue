<!--
 Copyright (c) 2020-2022 - for information on the respective copyright owner
 see the NOTICE file and/or the repository at
 https://github.com/hyperledger-labs/business-partner-agent

 SPDX-License-Identifier: Apache-2.0
-->
<template>
  <v-container>
    <v-card class="mx-auto">
      <v-card-title class="bg-light">{{
        $t("component.issueOobCredential.title")
      }}</v-card-title>
      <v-card-text>
        <v-stepper v-model="currentStep" flat outlined>
          <v-stepper-header>
            <v-stepper-step :complete="currentStep > 1" step="1">
              Settings
            </v-stepper-step>
            <v-divider></v-divider>
            <v-stepper-step :complete="currentStep > 2" step="2">
              QR Code / URL
            </v-stepper-step>
          </v-stepper-header>
          <v-stepper-items>
            <v-stepper-content step="1">
              <br />
              <v-select
                :label="$t('component.issueOobCredential.credDefLabel')"
                return-object
                v-model="credDef"
                :items="credDefList"
                outlined
                :disabled="this.$props.credDefId !== undefined"
                dense
                @change="credDefSelected"
              ></v-select>
              <v-card v-if="credDefLoaded && expertMode" class="my-4" outlined>
                <v-card-title class="bg-light" style="font-size: small"
                  >{{ $t("component.issueCredential.expertLoad.title") }}
                  <v-btn
                    icon
                    @click="expertLoad.show = !expertLoad.show"
                    style="margin-left: auto"
                  >
                    <v-icon v-if="expertLoad.show">$vuetify.icons.up</v-icon>
                    <v-icon v-else>$vuetify.icons.down</v-icon>
                  </v-btn>
                </v-card-title>
                <v-expand-transition>
                  <div v-show="expertLoad.show">
                    <v-card-text>
                      <v-row>
                        <v-textarea
                          rows="5"
                          outlined
                          dense
                          v-model="expertLoad.data"
                          :placeholder="
                            $t(
                              'component.issueCredential.expertLoad.dataPlaceholder'
                            )
                          "
                        ></v-textarea>
                      </v-row>
                      <v-row>
                        <v-file-input
                          v-model="expertLoad.file"
                          :label="
                            $t(
                              'component.issueCredential.expertLoad.filePlaceholder'
                            )
                          "
                          outlined
                          dense
                          @change="uploadExpertLoadFile"
                          :accept="expertLoad.fileAccept"
                          prepend-icon="$vuetify.icons.attachment"
                        ></v-file-input>
                      </v-row>
                    </v-card-text>
                    <v-card-actions>
                      <v-layout align-start justify-start>
                        <v-radio-group
                          v-model="expertLoad.type"
                          row
                          @change="expertLoadTypeChanged"
                        >
                          <v-radio label="JSON" value="json"></v-radio>
                          <v-radio label="CSV" value="csv"></v-radio>
                        </v-radio-group>
                      </v-layout>
                      <v-layout align-end justify-end>
                        <v-bpa-button
                          color="secondary"
                          @click="clearExpertLoad()"
                          :disabled="!expertLoadEnabled"
                          >{{
                            $t(
                              "component.issueCredential.expertLoad.buttons.clear"
                            )
                          }}
                        </v-bpa-button>
                        <v-bpa-button
                          color="primary"
                          @click="parseExpertLoad()"
                          :disabled="!expertLoadEnabled"
                          >{{
                            $t(
                              "component.issueCredential.expertLoad.buttons.load"
                            )
                          }}
                        </v-bpa-button>
                      </v-layout>
                    </v-card-actions>
                  </div>
                </v-expand-transition>
              </v-card>
              <v-card v-if="credDefLoaded" outlined>
                <v-card-title class="bg-light" style="font-size: small">{{
                  $t("component.issueOobCredential.attributesTitle")
                }}</v-card-title>
                <v-card-text>
                  <v-row>
                    <v-col>
                      <v-text-field
                        v-for="field in credDef.fields"
                        :key="field.type"
                        :label="field.label"
                        v-model="credentialFields[field.type]"
                        outlined
                        dense
                        @blur="enableSubmit"
                        @keyup="enableSubmit"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-card-text>
              </v-card>
              <v-card class="my-4" outlined>
                <v-card-title class="bg-light" style="font-size: small">
                  {{ $t("component.issueOobCredential.options.title") }}
                </v-card-title>
                <v-card-text v-if="expertMode">
                  <v-col>
                    <v-list-item>
                      <v-list-item-content>
                        <v-list-item-title
                          class="grey--text text--darken-2 font-weight-medium"
                        >
                          {{ $t("view.addPartner.setName") }}
                        </v-list-item-title>
                      </v-list-item-content>
                      <v-list-item-action>
                        <v-text-field
                          :label="$t('view.addPartnerbyURL.labelName')"
                          v-model="alias"
                          outlined
                          dense
                        >
                        </v-text-field>
                      </v-list-item-action>
                    </v-list-item>
                  </v-col>
                  <v-col>
                    <v-list-item>
                      <v-list-item-content>
                        <v-list-item-title
                          class="grey--text text--darken-2 font-weight-medium"
                        >
                          {{ $t("view.addPartner.setTags") }}
                        </v-list-item-title>
                      </v-list-item-content>
                      <v-list-item-action>
                        <v-autocomplete
                          multiple
                          v-model="selectedTags"
                          :items="tags"
                          chips
                          deletable-chips
                        >
                        </v-autocomplete>
                      </v-list-item-action>
                    </v-list-item>
                  </v-col>
                  <v-col>
                    <v-list-item>
                      <v-list-item-content>
                        <v-list-item-title
                          class="grey--text text--darken-2 font-weight-medium"
                          >{{
                            $t("view.addPartner.trustPing")
                          }}</v-list-item-title
                        >
                      </v-list-item-content>
                      <v-list-item-action>
                        <v-switch v-model="trustPing"></v-switch>
                      </v-list-item-action>
                    </v-list-item>
                  </v-col>
                </v-card-text>
              </v-card>
            </v-stepper-content>
            <v-stepper-content step="2">
              <qrcode-vue
                class="d-flex justify-center"
                :value="invitationUrl"
                :size="400"
                level="H"
              ></qrcode-vue>
              <v-expansion-panels class="mt-4">
                <v-expansion-panel>
                  <v-expansion-panel-header>
                    <span class="grey--text text--darken-2 font-weight-medium">
                      {{ $t("component.issueOobCredential.urlLabel") }}</span
                    >
                  </v-expansion-panel-header>
                  <v-expansion-panel-content>
                    <span class="font-weight-light">{{ invitationUrl }}</span>
                  </v-expansion-panel-content>
                </v-expansion-panel>
              </v-expansion-panels>
            </v-stepper-content>
          </v-stepper-items>
        </v-stepper>
      </v-card-text>
      <v-card-actions>
        <v-layout align-end justify-end>
          <v-bpa-button
            v-show="currentStep === 1"
            color="secondary"
            @click="cancel()"
            >{{ $t("button.cancel") }}</v-bpa-button
          >
          <v-bpa-button
            v-show="currentStep === 1"
            :loading="this.isBusy"
            color="primary"
            @click="submit()"
            :disabled="createDisabled"
            >{{ $t("button.create") }}</v-bpa-button
          >
          <v-bpa-button
            v-show="currentStep === 2"
            :loading="this.isBusy"
            color="primary"
            @click="$emit('success')"
            >{{ $t("button.close") }}</v-bpa-button
          >
        </v-layout>
      </v-card-actions>
    </v-card>
  </v-container>
</template>

<script lang="ts">
import Vue from "vue";
import { EventBus } from "@/main";
import {
  ApiCreateInvitation,
  issuerService,
  IssueOobCredentialRequest,
} from "@/services";
import VBpaButton from "@/components/BpaButton";
import * as textUtils from "@/utils/textUtils";
import * as CSV from "csv-string";
import QrcodeVue from "qrcode.vue";

export default {
  name: "IssueCredential",
  components: { VBpaButton, QrcodeVue },
  props: {
    credDefId: String,
    open: Boolean,
  },
  mounted() {
    this.load();
  },
  data: () => {
    return {
      isLoading: false,
      isBusy: false,
      credDef: {},
      credential: {},
      credentialFields: {},
      currentStep: 1,
      alias: "",
      selectedTags: [],
      // Disable trust ping for invitation to
      // mobile wallets by default.
      trustPing: false,
      invitationUrl: "",
      createDisabled: true,
      expertLoad: {
        show: false,
        data: "",
        file: undefined,
        type: "json",
        fileAccept: "text/plain,application/json",
      },
    };
  },
  computed: {
    expertMode() {
      return this.$store.state.expertMode;
    },
    credDefList: {
      get() {
        return this.$store.getters.getCredDefSelectList;
      },
    },
    credDefLoaded: {
      get() {
        return this.credDef?.fields?.length;
      },
    },
    expertLoadEnabled() {
      return this.expertLoad.data?.trim().length > 0;
    },
    tags() {
      return this.$store.state.tags
        ? this.$store.state.tags.map((tag) => tag.name)
        : [];
    },
  },
  watch: {
    credDefId(value) {
      if (value) {
        this.credDef = this.credDefList.find((p) => p.value === value);
        this.credDefSelected();
      }
    },
    open(value) {
      if (value) {
        // load up cred def (if needed)
        // this will happen if the form was opened with credDefId and then is cancelled and re-opened with the same credDefId
        // the credDef is empty and won't initialize unless credDefId changes.
        if (!this.credDef?.fields) {
          this.credDef = this.credDefList.find(
            (p) => p.value === this.$props.credDefId
          );
          this.credDefSelected();
        }
        this.clearExpertLoad();
        this.expertLoad.show = false;
      }
    },
  },
  methods: {
    async load() {
      this.isLoading = true;
      this.credDef = {};

      if (this.$props.credDefId) {
        this.credDef = this.credDefList.find(
          (c) => c.value === this.$props.credDefId
        );
      }

      this.isLoading = false;
    },
    async issueOobCredentialOffer(): Promise<ApiCreateInvitation> {
      // create an empty document, all empty strings
      let document: any = {};
      for (const x of this.credDef.fields) document[x.type] = "";
      // fill in whatever populated fields we have
      Object.assign(document, this.credentialFields);

      const data: IssueOobCredentialRequest = {
        alias: this.alias,
        tag: this.selectedTags,
        trustPing: this.trustPing,
        credDefId: this.credDef.id,
        document: document,
      };
      try {
        const resp = await issuerService.issueOobCredentialOfferCreate(data);

        if (resp.status === 200) {
          return resp.data;
        }
      } catch (error) {
        EventBus.$emit("error", this.$axiosErrorMessage(error));
      }
    },
    async submit() {
      this.isBusy = true;
      try {
        const oobInvitation: ApiCreateInvitation =
          await this.issueOobCredentialOffer();

        this.isBusy = false;

        if (oobInvitation) {
          EventBus.$emit(
            "success",
            this.$t("component.issueOobCredential.successMessage")
          );

          this.invitationUrl = oobInvitation.invitationUrl;
          this.credDef = {};
          this.createDisabled = true;
          this.currentStep = 2;
        }
      } catch (error) {
        this.isBusy = false;
        EventBus.$emit("error", this.$axiosErrorMessage(error));
      }
    },
    cancel() {
      // clear out selected credential definition, will select (or have pre-populated) when re-open form.
      this.credDef = {};
      this.credentialFields = {};
      this.alias = "";
      this.selectedTags = [];
      this.trustPing = false;
      this.clearExpertLoad();
      this.currentStep = 1;
      this.$emit("cancelled");
    },
    credDefSelected() {
      this.credentialFields = {};
      for (const x of this.credDef.fields)
        Vue.set(this.credentialFields, x.type, "");
      this.createDisabled = true;
    },
    enableSubmit() {
      let enabled = false;
      if (
        this.credDef &&
        this.credDef.fields &&
        this.credDef.fields.length > 0
      ) {
        console.log(this.credentialFields);
        enabled = this.credDef.fields.some(
          (x) =>
            this.credentialFields[x.type] &&
            this.credentialFields[x.type]?.trim().length > 0
        );
      }
      this.createDisabled = !enabled;
    },
    expertLoadTypeChanged(value) {
      this.expertLoad.fileAccept =
        value === "json"
          ? "text/plain,application/json"
          : "text/plain,text/csv";
    },
    uploadExpertLoadFile(v) {
      this.expertLoad.file = v;
      if (v) {
        try {
          let reader = new FileReader();
          reader.readAsText(v, "UTF-8");
          reader.addEventListener("load", (event_) => {
            this.expertLoad.data = event_.target.result;
          });
          reader.addEventListener("error", () => {
            EventBus.$emit(
              "error",
              `${this.$t(
                "component.issueCredential.expertLoad.errorMessages.readFile"
              )} '${v.name}'.`
            );
          });
        } catch (error) {
          EventBus.$emit(
            "error",
            `${this.$t(
              "component.issueCredential.expertLoad.errorMessages.readFile"
            )} '${v.name}'. ${error.message}`
          );
        }
      }
    },
    clearExpertLoad() {
      this.expertLoad = {
        show: true,
        data: "",
        file: undefined,
        type: "json",
        fileAccept: "text/plain,application/json",
      };
    },
    parseExpertLoad() {
      if (this.expertLoad.data) {
        let object;
        let formatErrorMessage = this.$t(
          "component.issueCredential.expertLoad.errorMessages.format.json"
        );
        if (this.expertLoad.type === "json") {
          object = this.jsonToObject(this.expertLoad.data);
        } else {
          formatErrorMessage = this.$t(
            "component.issueCredential.expertLoad.errorMessages.format.csv"
          );
          object = this.csvToObject(this.expertLoad.data);
        }

        if (object) {
          let count = 0;
          // see if we can populate the credential fields...
          for (const x of this.credDef.fields) {
            if (
              object[x.type] &&
              !(
                Object.prototype.toString.call(object[x.type]) ===
                  "[object Object]" ||
                Object.prototype.toString.call(object[x.type]) ===
                  "[object Function]"
              )
            ) {
              count = count + 1;
              Vue.set(this.credentialFields, x.type, object[x.type].toString());
            }
          }
          if (count) {
            this.enableSubmit();
          } else {
            EventBus.$emit(
              "error",
              this.$t(
                "component.issueCredential.expertLoad.errorMessages.attributes"
              )
            );
          }
        } else {
          let errorMessage = this.$t(
            "component.issueCredential.expertLoad.errorMessages.parse"
          );
          EventBus.$emit("error", `${errorMessage} ${formatErrorMessage}`);
        }
      }
    },
    jsonToObject(data) {
      let object;
      if (data && Object.prototype.toString.call(data) === "[object String]") {
        try {
          object = JSON.parse(data);
        } catch {
          console.log("Error converting JSON string to Object");
        }
      }
      return object;
    },
    csvToObject(data) {
      let object;
      if (data && Object.prototype.toString.call(data) === "[object String]") {
        try {
          const array = CSV.parse(data);
          if (array?.length > 1) {
            const names = array[0];
            const values = array[1]; // only grab first row for now...
            const namesOk = names.every((value) =>
              textUtils.isValidSchemaAttributeName(value)
            );
            if (namesOk) {
              object = {};
              for (const [index, a] of names.entries()) {
                object[a] = values[index];
              }
            }
          }
        } catch {
          console.log("Error converting CSV string to Object");
        }
      }
      return object;
    },
  },
};
</script>
