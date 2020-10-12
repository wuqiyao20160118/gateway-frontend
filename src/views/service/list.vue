<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="listQuery.info"
        placeholder="Service Name / Service Discription"
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
      <router-link :to="'/service/service_create_http/'">
        <el-button
          class="filter-item"
          style="margin-left: 10px"
          type="primary"
          icon="el-icon-edit"
        >
          Add HTTP Service
        </el-button>
      </router-link>
      <router-link :to="'/service/service_create_tcp/'">
        <el-button
          class="filter-item"
          style="margin-left: 10px"
          type="primary"
          icon="el-icon-edit"
        >
          Add TCP Service
        </el-button>
      </router-link>
      <router-link :to="'/service/service_create_grpc/'">
        <el-button
          class="filter-item"
          style="margin-left: 10px"
          type="primary"
          icon="el-icon-edit"
        >
          Add GRPC Service
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
      <el-table-column label="Service Name" min-width="100px">
        <template slot-scope="{ row }">
          <span>{{ row.service_name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Service Discription" min-width="120px">
        <template slot-scope="{ row }">
          <span>{{ row.service_desc }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Service Type" min-width="70px">
        <template slot-scope="{ row }">
          <span>{{ row.load_type | loadTypeFilter }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Service Address" min-width="160px">
        <template slot-scope="{ row }">
          <span>{{ row.service_addr }}</span>
        </template>
      </el-table-column>
      <el-table-column label="QPS" min-width="70px">
        <template slot-scope="{ row }">
          <span>{{ row.qps }}</span>
        </template>
      </el-table-column>
      <el-table-column label="QPD" min-width="80px">
        <template slot-scope="{ row }">
          <span>{{ row.qpd }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Node Number" min-width="60px">
        <template slot-scope="{ row }">
          <span>{{ row.total_node }}</span>
        </template>
      </el-table-column>

      <el-table-column
        label="Actions"
        align="center"
        min-width="90"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <router-link :to="'/service/service_stat/' + row.id">
            <el-button type="primary" size="mini"> Analyze </el-button>
          </router-link>
          <router-link
            v-if="rwo.load_type === 0"
            :to="'/service/service_edit_http/' + row.id"
          >
            <el-button type="primary" size="mini"> Edit </el-button>
          </router-link>
          <router-link
            v-if="rwo.load_type === 1"
            :to="'/service/service_edit_tcp/' + row.id"
          >
            <el-button type="primary" size="mini"> Edit </el-button>
          </router-link>
          <router-link
            v-if="rwo.load_type === 2"
            :to="'/service/service_edit_grpc/' + row.id"
          >
            <el-button type="primary" size="mini"> Edit </el-button>
          </router-link>
          <el-button
            v-if="row.status != 'deleted'"
            size="mini"
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
import { serviceList, serviceDelete } from "@/api/service";
import waves from "@/directive/waves"; // waves directive
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
import { title } from "@/settings";

const loadTypeOptions = [
  { key: "0", display_name: "HTTP" },
  { key: "1", display_name: "TCP" },
  { key: "2", display_name: "GRPC" },
];

// arr to obj, such as { CN : "China", US : "USA" }
const loadTypeKeyValue = loadTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name;
  return acc;
}, {});

export default {
  name: "service_list",
  components: { Pagination },
  directives: { waves },
  filters: {
    loadTypeFilter(type) {
      return loadTypeKeyValue[type];
    },
  },
  data() {
    return {
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page_no: 1,
        page_size: 20,
        info: "",
      },
      temp: {
        id: undefined,
      },
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      this.listLoading = true;
      serviceList(this.listQuery).then((response) => {
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
          serviceDelete(deleteQuery).then((response) => {
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

