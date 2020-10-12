<template>
  <div class="mixin-components-container">
    <el-row>
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <span v-if="isEdit === false">Create GRPC Service</span>
          <span v-if="isEdit === true">Edit GRPC Service</span>
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
            <el-form-item label="Port" class="is-required">
              <el-input
                placeholder="The input should be an integer between 8001-8999, required"
                v-model="form.port"
                :disabled="isEdit === true"
              ></el-input>
            </el-form-item>
            <el-form-item label="Metadata Transformation">
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
            <el-form-item>
              <el-button
                type="primary"
                @click="onSubmit"
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
  serviceAddGrpc,
  serviceDetail,
  serviceUpdateGrpc,
} from "@/api/service";

export default {
  name: "ServiceAddGrpc",

  data() {
    return {
      isEdit: false,
      submitButtonDisabled: false,
      form: {
        id: "0",
        service_name: "",
        service_desc: "",
        port: "",
        header_transfor: "",
        round_type: 2,
        ip_list: "",
        weight_list: "",
        open_auth: 0,
        black_list: "",
        white_list: "",
        clientip_flow_limit: "",
        service_flow_limit: "",
      },
    };
  },

  methods: {
    onSubmit() {
      this.submitButtonDisabled = true;
      const query = Object.assign({}, this.form);
      console.log(query);

      query.white_list = query.white_list.replace(/\n/g, ",");
      query.black_list = query.black_list.replace(/\n/g, ",");
      query.weight_list = query.weight_list.replace(/\n/g, ",");
      query.header_transfor = query.header_transfor.replace(/\n/g, ",");
      query.ip_list = query.ip_list.replace(/\n/g, ",");

      query.clientip_flow_limit = Number(query.clientip_flow_limit);
      query.service_flow_limit = Number(query.service_flow_limit);
      query.port = Number(query.port);

      if (this.isEdit) {
        serviceUpdateGrpc(query)
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
        serviceAddGrpc(query)
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
        this.form.port = response.data.tcp_rule.port.toString();

        this.form.round_type = response.data.load_balance.round_type;
        this.form.ip_list = response.data.load_balance.ip_list.replace(
          /, /g,
          "\n"
        );
        this.form.weight_list = response.data.load_balance.weight_list.replace(
          /, /g,
          "\n"
        );
        this.form.header_transfor = response.data.grpc_rule.header_transfor.replace(
          /,/g,
          "\n"
        );

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
.el-select .el-input {
  width: 130px;
}
.input-with-select .el-input-group__prepend {
  background-color: #fff;
}
</style>
