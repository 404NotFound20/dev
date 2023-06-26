<template>
  <el-card class="box-card1">
    <el-card class="box-card1">
      <el-form label-position="left" inline>
        <el-form-item label="查询场景" label-position="left">
          <el-cascader :options="options" v-model="selected" @change="queryConfigApi"></el-cascader>
        </el-form-item>
        <el-form-item><el-switch style="display: block" v-model="showConfig" active-color="#13ce66"
            inactive-color="#ff4949" active-text="是否展示配置详情">
          </el-switch></el-form-item>
      </el-form>
    </el-card>
    <el-card v-if="showConfig" class="box-card1">
      <el-table :data="configData" border stripe max-height="600">
        <el-table-column prop="business" label="业务类型"></el-table-column>
        <el-table-column prop="scenario" label="业务场景"></el-table-column>
        <el-table-column prop="dbName" label="库名" width="300px"></el-table-column>
        <el-table-column prop="ifDb" label="分库"><template slot-scope="scope">
            <el-tag v-show="scope.row.ifDb" type="success">是</el-tag>
            <el-tag v-show="!scope.row.ifDb" type="info">否</el-tag>
          </template>
          <template slot="header" slot-scope="{ column }">
            <el-tooltip effect="dark" content="仅当selectKey为ssoid、userid时可生效" placement="top">
              <span>{{ column.label }} ！</span>
            </el-tooltip>
          </template>
        </el-table-column>
        <el-table-column prop="tbName" label="表名" width="400px"></el-table-column>
        <el-table-column prop="ifTable" label="分表"><template slot-scope="scope">
            <el-tag v-show="scope.row.ifTable" type="success">是</el-tag>
            <el-tag v-show="!scope.row.ifTable" type="info">否</el-tag>
          </template>
          <template slot="header" slot-scope="{ column }">
            <el-tooltip effect="dark" content="仅当selectKey为ssoid、userid时可生效" placement="top">
              <span>{{ column.label }} ！</span>
            </el-tooltip>
          </template>
        </el-table-column>
        <el-table-column prop="selectKey" label="查询键"></el-table-column>
        <el-table-column prop="tbSelectKey" label="真实键">
          <template slot="header" slot-scope="{ column }">
            <el-tooltip effect="dark" content="数据库真实的查询字段，下方左侧" placement="top">
              <span>{{ column.label }} ！</span>
            </el-tooltip>
          </template>
          <template slot-scope="{ row }">
            <span>{{ row.tbSelectKey }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="filtration" label="过滤字段">
          <template slot="header" slot-scope="{ column }">
            <el-tooltip effect="dark" content="当前表支持的过滤字段，下方右侧" placement="top">
              <span>{{ column.label }} ！</span>
            </el-tooltip>
          </template>
          <template slot-scope="{ row }">
            <span>{{ row.filtration }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="decision" label="描述"></el-table-column>
        <el-table-column prop="effective" label="生效"><template slot-scope="scope">
            <el-tag v-show="scope.row.effective" type="success">是</el-tag>
            <el-tag v-show="!scope.row.effective" type="info">否</el-tag>
          </template></el-table-column>
      </el-table>
    </el-card>
    <el-card class="box-card1">
      <div style="display: flex;">
        <div style="flex: 1;">
          <h3>查询主键（必填）</h3>
          <el-form label-position="left">
            <el-form-item v-for="key in selectKeys" :key="key" :label="key">
              <el-input v-model="inputMap[key]" style="width: 400px; margin-left: 10px;"></el-input>
            </el-form-item>
          </el-form>
        </div>
        <div style="flex: 1;">
          <h3>过滤字段（可不填）</h3>
          <el-form label-position="left">
            <el-form-item v-for="key in filtrationKeys" :key="key" :label="key">
              <el-input v-model="inputMap[key]" style="width: 400px; margin-left: 10px;"></el-input>
            </el-form-item>
          </el-form>
        </div>
      </div>
      <el-button icon="el-icon-search" type="primary" plain @click="queryApi">查询结果</el-button>
    </el-card>
    <el-card class="box-card1">
    <el-radio-group v-model="viewMode">
      <el-radio label="Table">表格</el-radio>
      <el-radio label="Transpose">转置表</el-radio>
      <el-radio label="JSON">JSON</el-radio>
    </el-radio-group>
    <el-tabs>
      <el-tab-pane v-for="(tableData, tableName) in resData" :key="tableName" :label="tableName">
        <el-button @click="queryColumnCommentApi(tableName)" type="primary" plain size="mini">展示字段描述</el-button>

        <div v-if="viewMode === 'Table'">
          <el-table :data="tableData" height="800" border stripe>
            <el-table-column v-for="(value, key) in tableData[0]" :key="key" :prop="key" :label="key"
              show-overflow-tooltip sortable></el-table-column>
          </el-table>
        </div>
        <div v-else-if="viewMode === 'JSON'">
          <pre>{{ JSON.stringify(tableData, null, 2) }}</pre>
        </div>
        <div v-else>
          <el-table :data="transposedData(tableData)" height="800" border stripe>
            <el-table-column v-for="(value, key) in transposedData(tableData)[0]" :key="key" :prop="key" :label="key"
              show-overflow-tooltip sortable></el-table-column>
          </el-table>
        </div>
      </el-tab-pane>
    </el-tabs>
  </el-card>
  </el-card>
</template>

<script>
import { query } from "../../../api/financeCredit";
import { queryConfig } from "../../../api/financeCredit";
import { queryConfigLabel } from "../../../api/financeCredit";
import { queryColumnComment } from "../../../api/financeCredit";
import { updateConfig } from "../../../api/financeCredit";
import { getEnv } from "@/utils/auth";

export default {
  data() {
    return {
      viewMode: 'Table',
      transpose: false,
      showConfig: false,
      inputMap: {},
      selected: [],
      labelMap: [],
      configData: [],
      resData: [],
      columnComment: [],
    };
  },
  computed: {
    selectKeys() {
      return [...new Set(this.configData.map((item) => item.selectKey))];
    },
    filtrationKeys() {
      let filtrationSet = new Set();
      this.configData.forEach((item) => {
        if (item.filtration) {
          let filtrations = item.filtration.split(",");
          filtrations.forEach((filtration) => {
            filtrationSet.add(filtration);
          });
        }
      });
      return filtrationSet;
    },
    options() {
      return Object.keys(this.labelMap).map((key) => ({
        value: key,
        label: key,
        children: this.labelMap[key].map((item) => ({
          value: item,
          label: item,
        })),
      }));
    },
  },
  methods: {
    transposedData(data) {
      const keys = Object.keys(data[0]);
      return keys.map((key) => {
        const row = { Field: key };
        data.forEach((item, index) => {
          row[`Column ${index + 1}`] = item[key];
        });
        return row;
      });
    },
    async queryApi() {
      const config = {
        business: this.selected[0],
        scenario: this.selected[1],
        env: getEnv(),
        keyMap: this.inputMap,
      };
      query(config).then((response) => {
        if (response.code === "0000") {
          this.resData = response.data;
        }
      });
    },
    async queryColumnCommentApi(Name) {
      console.log(Name);
      const config = {
        env: getEnv(),
        dbName: Name.split(".")[0],
        tableName: Name.split(".")[1],
      };
      queryColumnComment(config).then((response) => {
        if (response.code === "0000") {
          this.columnComment = response.data;
          this.resData[Name].unshift(this.columnComment);
        }
      });
    },
    async queryConfigApi() {
      const config = {
        business: this.selected[0],
        scenario: this.selected[1],
      };
      queryConfig(config).then((response) => {
        if (response.code === "0000") {
          this.configData = response.data;
        }
      });
    },
    queryConfigLabelApi() {
      console.log("test");
      queryConfigLabel().then((response) => {
        if (response.code === "0000") {
          this.labelMap = response.data;
        }
      });
    },
  },
  created: function () {
    this.queryConfigLabelApi();
  },
};
</script>

<style>
.box-card1 {
  margin-top: 0px;
  margin-left: 0px;
  width: 100%;
}
</style>