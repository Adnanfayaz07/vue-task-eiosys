<template>
  <div>
    <div v-if="step === 1">
      <label for="numColumns">Number of Columns:</label>
      <input type="number" id="numColumns" v-model="numColumns" />
      <button @click="defineColumns">Define Columns</button>
    </div>
    <div v-else-if="step === 2">
      <div v-for="(column, index) in columns" :key="index">
        <label :for="'colName' + index">Column {{ index + 1 }} Name:</label>
        <input type="text" :id="'colName' + index" v-model="column.name" />
        <label :for="'colType' + index">Data Type:</label>
        <select :id="'colType' + index" v-model="column.type">
          <option v-for="type in dataTypes" :key="type" :value="type">{{ type }}</option>
        </select>
      </div>
      <button @click="createTable">Create Table</button>
    </div>
    <div v-else>
      <div class="table-controls">
        <el-select v-model="selectedColumns" multiple placeholder="Columns">
          <el-option
            v-for="column in columns"
            :key="column.name"
            :label="column.name"
            :value="column.name">
          </el-option>
        </el-select>
      </div>
      <el-table :data="rows" style="width: 100%" stripe border>
        <el-table-column
          v-for="(column, index) in displayedColumns"
          :key="index"
          :prop="column.name"
          :label="column.name"
          width="150">
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      step: 1,
      numColumns: 0,
      columns: [],
      rows: [
        { id: 1, customerName: 'Alice Johnson', city: 'New York', email: 'alice.johnson@example.com', phoneNumber: '(212) 555-1234' },
        { id: 2, customerName: 'Bob Smith', city: 'Los Angeles', email: 'bob.smith@example.com', phoneNumber: '(310) 555-5678' },
        { id: 3, customerName: 'Carol Davis', city: 'Chicago', email: 'carol.davis@example.com', phoneNumber: '(312) 555-9101' },
        { id: 4, customerName: 'David Wilson', city: 'Houston', email: 'david.wilson@example.com', phoneNumber: '(713) 555-1122' },
        { id: 5, customerName: 'Eva Martinez', city: 'Phoenix', email: 'eva.martinez@example.com', phoneNumber: '(602) 555-3344' },
        { id: 6, customerName: 'Frank Lee', city: 'Philadelphia', email: 'frank.lee@example.com', phoneNumber: '(215) 555-5566' },
        { id: 7, customerName: 'Grace Brown', city: 'San Antonio', email: 'grace.brown@example.com', phoneNumber: '(210) 555-7788' },
        { id: 8, customerName: 'Hank Taylor', city: 'San Diego', email: 'hank.taylor@example.com', phoneNumber: '(858) 555-9900' },
        { id: 9, customerName: 'Ivy Anderson', city: 'Dallas', email: 'ivy.anderson@example.com', phoneNumber: '(214) 555-1212' },
        { id: 10, customerName: 'Jack Thomas', city: 'San Jose', email: 'jack.thomas@example.com', phoneNumber: '(408) 555-3434' }
      ],
      selectedColumns: ['id', 'customerName', 'city', 'email', 'phoneNumber'], // Initialize selected columns
      dataTypes: [
        "text", "number", "password", "email", "date",
        "datetime-local", "month", "week", "time", "url",
        "tel", "color"
      ]
    };
  },
  computed: {
    displayedColumns() {
      return this.columns.filter(column => this.selectedColumns.includes(column.name));
    }
  },
  methods: {
    defineColumns() {
      this.columns = [];
      for (let i = 0; i < this.numColumns; i++) {
        this.columns.push({ name: "", type: "text", display: true });
      }
      this.step = 2;
    },
    createTable() {
      this.step = 3;
    },
    addRow() {
      const newRow = {};
      this.columns.forEach(column => {
        newRow[column.name] = "";
      });
      this.rows.push(newRow);
    }
  }
};
</script>

<style scoped>
.table-controls {
  margin-bottom: 20px;
}
</style>
