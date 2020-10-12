<template>
  <div class="mixin-components-container">
    <el-row>
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <span v-if="isEdit === false">Create User</span>
          <span v-if="isEdit === true">Edit User</span>
        </div>
        <div style="margin-bottom: 50px">
          <el-form ref="form" :model="form" label-width="140px">
            <el-form-item label="User App_id" class="is-required">
              <el-input
                placeholder="6-128 characters"
                v-model="form.app_id"
                :disabled="isEdit === true"
              ></el-input>
            </el-form-item>
            <el-form-item label="User Name" class="is-required">
              <el-input
                placeholder="0-255 characters, required"
                v-model="form.name"
              ></el-input>
            </el-form-item>
            <el-form-item label="32bit Secret Key">
              <el-input
                v-model="form.secret"
                placeholder="32bit Secret Key，not required，auto-generated"
              />
            </el-form-item>
            <el-form-item label="QPS Limitation">
              <el-input
                placeholder="0: no limitation"
                v-model="form.qps"
              ></el-input>
            </el-form-item>
            <el-form-item label="QPD Limitation">
              <el-input
                placeholder="0: no limitation"
                v-model="form.qpd"
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
import { appAdd, appDetail, appUpdate } from "@/api/app";

export default {
  name: "AddAppUser",

  data() {
    return {
      isEdit: false,
      submitButtonDisabled: false,
      form: {
        id: "0",
        app_id: "",
        name: "",
        secret: "",
        white_ips: "",
        qpd: "",
        qps: "",
      },
    };
  },

  methods: {
    onSubmit() {
      this.submitButtonDisabled = true;
      const query = Object.assign({}, this.form);
      console.log(query);

      query.id = Number(query.id);
      query.qpd = Number(query.qpd);
      query.qps = Number(query.qps);

      if (this.isEdit) {
        appUpdate(query)
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
        appAdd(query)
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
      appDetail(query).then((response) => {
        this.form = response.data;
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