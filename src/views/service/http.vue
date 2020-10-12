<template>
  <div class="mixin-components-container">
    <el-row>
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <span v-if="isEdit === false">Create HTTP Service</span>
          <span v-if="isEdit === true">Edit HTTP Service</span>
        </div>
        <div style="margin-bottom: 50px">
          <el-form ref="form" :model="form" label-width="140px">
            <el-form-item label="Service Name" class="is-required">
              <el-input
                placeholder="6-128 characters"
                v-model="form.service_name"
                :disabled="isEdit === true"
              ></el-input>
            </el-form-item>
            <el-form-item label="Service Discription" class="is-required">
              <el-input
                placeholder="0-255 characters, required"
                v-model="form.service_desc"
              ></el-input>
            </el-form-item>
            <el-form-item label="Service Type" class="is-required">
              <el-input
                placeholder="Path format: /user/, Domain format: www.test.com"
                v-model="form.rule"
                class="input-with-select"
                :disabled="isEdit === true"
              >
                <el-select
                  v-model="form.rule_type"
                  slot="prepend"
                  placeholder="Please Select"
                  style="width=80px"
                  :disabled="isEdit === true"
                >
                  <el-option label="Path" :value="0"></el-option>
                  <el-option label="Domain" :value="1"></el-option>
                </el-select>
              </el-input>
            </el-form-item>
            <el-form-item label="Support HTTPS">
              <el-switch
                v-model="form.need_https"
                :active-value="1"
                :inactive-value="0"
              />
              <span style="color: #606266; font-weight: 700"
                >&nbsp;&nbsp;&nbsp;Support Strip_URI&nbsp;&nbsp;</span
              >
              <el-switch
                v-model="form.need_strip_uri"
                :active-value="1"
                :inactive-value="0"
              />
              <span style="color: #606266; font-weight: 700"
                >&nbsp;&nbsp;&nbsp;Support Websocket&nbsp;&nbsp;</span
              >
              <el-switch
                v-model="form.need_websocket"
                :active-value="1"
                :inactive-value="0"
              />
            </el-form-item>
            <el-form-item label="URL Rewrite">
              <el-input
                type="textarea"
                placeholder="Format: ^/gatekeeper/test_service(.*) $1, multiple lines"
                v-model="form.url_rewrite"
                autosize
              ></el-input>
            </el-form-item>
            <el-form-item label="Header Transformation">
              <el-input
                type="textarea"
                placeholder="Support type: add/del/edit; Format: add headname headvalue; multiple lines"
                v-model="form.header_transfor"
                autosize
              ></el-input>
            </el-form-item>
            <el-form-item label="Enable Validation">
              <el-switch
                v-model="form.open_auth"
                :active-value="1"
                :inactive-value="0"
              ></el-switch>
            </el-form-item>
            <el-form-item label="IP Whitelist">
              <el-input
                type="textarea"
                placeholder="Format: 127.0.0.1:80, multiple lines, whitelist is prior to blacklist"
                v-model="form.white_list"
                autosize
              ></el-input>
            </el-form-item>
            <el-form-item label="IP Blacklist">
              <el-input
                type="textarea"
                placeholder="Multiple lines"
                v-model="form.black_list"
                autosize
              ></el-input>
            </el-form-item>
            <el-form-item label="Flow Limitation on ClientIP">
              <el-input
                placeholder="0: no limitation"
                v-model="form.clientip_flow_limit"
              ></el-input>
            </el-form-item>
            <el-form-item label="Flow Limitation on Service">
              <el-input
                placeholder="0: no limitation"
                v-model="form.service_flow_limit"
              ></el-input>
            </el-form-item>
            <el-form-item label="Round Type">
              <el-radio-group v-model="form.round_type">
                <el-radio label="0">random</el-radio>
                <el-radio label="1">round-robin</el-radio>
                <el-radio label="2">weight_round-robin</el-radio>
                <el-radio label="3">ip_hash</el-radio>
              </el-radio-group>
            </el-form-item>
            <el-form-item label="IP List" class="is-required">
              <el-input
                type="textarea"
                placeholder="Format: 127.0.0.1:80, multiple lines"
                v-model="form.ip_list"
                autosize
              ></el-input>
            </el-form-item>
            <el-form-item label="Weight List" class="is-required">
              <el-input
                type="textarea"
                placeholder="Multiple lines"
                v-model="form.weight_list"
                autosize
              ></el-input>
            </el-form-item>
            <el-form-item label="Upstream Connection Timeout">
              <el-input
                placeholder="unit:s, 0: no limitation"
                v-model="form.upstream_connect_timeout"
              ></el-input>
            </el-form-item>
            <el-form-item label="Upstream Header Timeout">
              <el-input
                placeholder="unit:s, 0: no limitation"
                v-model="form.upstream_header_timeout"
              ></el-input>
            </el-form-item>
            <el-form-item label="Max Idle Time">
              <el-input
                placeholder="unit:s, 0: no limitation"
                v-model="form.upstream_idle_timeout"
              ></el-input>
            </el-form-item>
            <el-form-item label="Max Idle Upstreams">
              <el-input
                placeholder="0: no limitation"
                v-model="form.upstream_max_idle"
              ></el-input>
            </el-form-item>
            <el-form-item>
              <el-button
                type="primary"
                @click="handleSubmit"
                :disabled="submitButtonDisabled"
                >Submit</el-button
              >
            </el-form-item>
          </el-form>
        </div>
      </el-card>
    </el-row>
  </div>
</template>

<script>
import {
  serviceAddHttp,
  serviceDetail,
  serviceUpdateHttp,
} from "@/api/service";

export default {
  name: "ServiceAddHttp",

  data() {
    return {
      isEdit: false,
      submitButtonDisabled: false,
      form: {
        service_name: "",
        service_desc: "",
        rule_type: 0,
        rule: "",
        need_https: 0,
        need_websocket: 0,
        need_strip_uri: 1,
        url_rewrite: "",
        header_transfor: "",
        round_type: 2,
        ip_list: "",
        weight_list: "",
        upstream_connect_timeout: "",
        upstream_header_timeout: "",
        upstream_idle_timeout: "",
        upstream_max_idle: "",
        open_auth: 0,
        black_list: "",
        white_list: "",
        clientip_flow_limit: "",
        service_flow_limit: "",
      },
    };
  },

  methods: {
    handleSubmit() {
      this.submitButtonDisabled = true;
      const query = Object.assign({}, this.form);
      console.log(query);

      query.white_list = query.white_list.replace(/\n/g, ",");
      query.black_list = query.black_list.replace(/\n/g, ",");
      query.weight_list = query.weight_list.replace(/\n/g, ",");
      query.url_rewrite = query.url_rewrite.replace(/\n/g, ",");
      query.header_transfor = query.header_transfor.replace(/\n/g, ",");
      query.ip_list = query.ip_list.replace(/\n/g, ",");

      query.clientip_flow_limit = Number(query.clientip_flow_limit);
      query.upstream_connect_timeout = Number(query.upstream_connect_timeout);
      query.upstream_header_timeout = Number(query.upstream_header_timeout);
      query.upstream_idle_timeout = Number(query.upstream_idle_timeout);
      query.upstream_max_idle = Number(query.upstream_max_idle);
      query.service_flow_limit = Number(query.service_flow_limit);

      if (this.isEdit) {
        serviceUpdateHttp(query)
          .then((response) => {
            this.submitButtonDisabled = false;
            this.$notify({
              title: "Success",
              message: "Edit successfully!",
              type: "success",
              duration: 2000,
            });
          })
          .catch(() => {
            this.submitButtonDisabled = false;
          });
      } else {
        serviceAddHttp(query)
          .then((response) => {
            this.submitButtonDisabled = false;
            this.$notify({
              title: "Success",
              message: "Add successfully!",
              type: "success",
              duration: 2000,
            });
          })
          .catch(() => {
            this.submitButtonDisabled = false;
          });
      }
    },

    fetchData(id) {
      const query = { id: id };
      serviceDetail(query).then((response) => {
        this.form.id = response.data.info.id;
        this.form.load_type = response.data.info.load_type;
        this.form.service_name = response.data.info.service_name;
        this.form.rule_type = response.data.http_rule.rule_type;
        this.form.rule = response.data.http_rule.rule;
        this.form.need_https = response.data.http_rule.need_https;
        this.form.need_websocket = response.data.http_rule.need_websocket;
        this.form.need_strip_uri = response.data.http_rule.need_strip_uri;
        this.form.url_rewrite = response.data.http_rule.url_rewrite.replace(
          /, /g,
          "\n"
        );
        this.form.header_transfor = response.data.http_rule.header_transfor.replace(
          /, /g,
          "\n"
        );
        this.form.round_type = response.data.load_balance.round_type;
        this.form.ip_list = response.data.load_balance.ip_list.replace(
          /, /g,
          "\n"
        );
        this.form.weight_list = response.data.load_balance.weight_list.replace(
          /, /g,
          "\n"
        );
        this.form.upstream_connect_timeout =
          response.data.load_balance.upstream_connect_timeout;
        this.form.upstream_header_timeout =
          response.data.load_balance.upstream_header_timeout;
        this.form.upstream_idle_timeout =
          response.data.load_balance.upstream_idle_timeout;
        this.form.upstream_max_idle =
          response.data.load_balance.upstream_max_idle;
        this.form.open_auth = response.data.access_control.open_auth;
        this.form.black_list = response.data.access_control.black_list.replace(
          /, /g,
          "\n"
        );
        this.form.white_list = response.data.access_control.white_list.replace(
          /, /g,
          "\n"
        );
        this.form.clientip_flow_limit =
          response.data.access_control.clientip_flow_limit;
        this.form.service_flow_limit =
          response.data.access_control.service_flow_limit;
      });
    },
  },

  created() {
    _;
    const id = this.$route.params && this.$route.params.id;
    if (id > 0) {
      this.isEdit = true;
      this.fetchData(id);
    }
  },
};
</script>

<style scoped>
.mixin-components-container {
  background-color: #f0f2f5;
  padding: 30px;
  min-height: calc(100vh - 84px);
}
.component-item {
  min-height: 100px;
}
</style>
