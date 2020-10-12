<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="listQuery.info"
        placeholder="App_id / User Name"
        style="width: 200px"
        class="filter-item"
        @keyup.enter.native="handleFilter"
      />
      <el-button
        v-waves
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        @click="handleFilter"
      >
        Search
      </el-button>
      <router-link :to="'/app/app_create'">
        <el-button
          class="filter-item"
          style="margin-left: 10px"
          type="primary"
          icon="el-icon-edit"
        >
          Add User
        </el-button>
      </router-link>
    </div>

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%"
    >
      <el-table-column label="ID" prop="id" align="center" width="50">
        <template slot-scope="{ row }">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="app_id" min-width="50px">
        <template slot-scope="{ row }">
          <span>{{ row.app_id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="User Name" min-width="80px">
        <template slot-scope="{ row }">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Serect" min-width="120px">
        <template slot-scope="{ row }">
          <span>{{ row.serect }}</span>
        </template>
      </el-table-column>
      <el-table-column label="QPS" min-width="100px">
        <template slot-scope="{ row }">
          <span>{{ row.real_qps }}</span>
        </template>
      </el-table-column>
      <el-table-column label="QPD" min-width="110px">
        <template slot-scope="{ row }">
          <span>{{ row.real_qpd }}</span>
        </template>
      </el-table-column>

      <el-table-column
        label="Actions"
        align="center"
        min-width="90"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row }">
          <router-link :to="'/app/app_stat/' + row.id">
            <el-button type="primary" size="small"> Analyze </el-button>
          </router-link>
          <router-link :to="'/app/app_edit/' + row.id">
            <el-button type="primary" size="small"> Edit </el-button>
          </router-link>
          <el-button
            v-if="row.status != 'deleted'"
            size="small"
            type="danger"
            @click="handleDelete(row, $index)"
          >
            Delete
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page_no"
      :limit.sync="listQuery.page_size"
      @pagination="getList"
    />
  </div>
</template>

<script>
import { appList, appDelete } from "@/api/app";
import waves from "@/directive/waves"; // waves directive
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
import { title } from "@/settings";

export default {
  name: "service_list",
  components: { Pagination },
  directives: { waves },
  filters: {},
  data() {
    return {
      tableKey: "qq",
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page_no: 1,
        page_size: 20,
        info: "",
      },
      parentsProps: {
        checkStrictly: true,
        value: "id",
        label: "name",
        children: "children",
      },
      parentsKey: 0,
      temp: {
        id: undefined,
        app_id: undefined,
        name: undefined,
        secret: undefined,
        white_ips: undefined,
        qpd: undefined,
        qps: undefined,
        created_at: undefined,
        updated_at: undefined,
      },
      dialogFormVisible: false,
      dialogStatus: "",
      rules: {},
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      this.listLoading = true;
      appList(this.listQuery).then((response) => {
        this.list = response.data.list;
        this.total = response.data.total;
        this.listLoading = false;
      });
    },
    handleFilter() {
      this.listQuery.page_no = 1;
      this.getList();
    },
    handleDelete(row, index) {
      this.$confirm(
        "Current operation will delete the record permanently, continue?",
        "Hint",
        {
          confirmButtonText: "Confirm",
          cancelButtonText: "Cancel",
          type: "warning",
        }
      )
        .then(() => {
          const deleteQuery = {
            id: row.id,
          };
          appDelete(deleteQuery).then((response) => {
            this.$notify({
              title: "Success",
              message: "Delete successfully!",
              type: "success",
              duration: 2000,
            });
            this.getList();
          });
        })
        .catch(() => {
          this.$notify({
            title: "Success",
            message: "Delete has been canceled!",
            type: "info",
            duration: 2000,
          });
        });
    },
  },
};
</script>

