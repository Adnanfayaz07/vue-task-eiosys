<template>
  <div>
    <!-- Step 1: Define Number of Columns and Column Details -->
    <div v-if="step === 1">
      <el-form :model="form">
        <el-form-item label="Number of Columns:">
          <el-input-number v-model="numColumns" @change="updateColumns" :min="1" controls-position="right"></el-input-number>
        </el-form-item>

        <div v-for="(column, index) in columns" :key="index" class="column-form">
          <el-form-item :label="'Column ' + (index + 1) + ' Name:'">
            <el-input v-model="column.name" placeholder="Column Name"></el-input>
          </el-form-item>
          <el-form-item :label="'Column ' + (index + 1) + ' Type:'">
            <el-select v-model="column.type" placeholder="Data Type">
              <el-option label="String" value="string"></el-option>
              <el-option label="Number" value="number"></el-option>
              <el-option label="Boolean" value="boolean"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item :label="'Column ' + (index + 1) + ' Dummy Value:'">
            <el-input v-model="column.value" placeholder="Dummy Value"></el-input>
          </el-form-item>
        </div>

        <el-button type="primary" @click="createTable">Create Table</el-button>
      </el-form>
    </div>

    <!-- Step 3: Display Table -->
    <div v-else>
      <el-table :data="tableData" style="width: 100%" :empty-text="'No data available'">
        <el-table-column
          v-for="(column, index) in columns"
          :key="index"
          :prop="column.name"
          :label="column.name"
        >
          <template slot-scope="scope">
            <el-input v-if="column.type === 'string'" v-model="scope.row[column.name]" placeholder="Enter text" />
            <el-input-number v-if="column.type === 'number'" v-model="scope.row[column.name]" placeholder="Enter number" disabled/>
            <el-switch v-if="column.type === 'boolean'" v-model="scope.row[column.name]" disabled/>
          </template>
        </el-table-column>
        <el-table-column label="Actions">
          <template slot-scope="scope">
            <el-button type="text" icon="el-icon-close" @click="removeRow(scope.row.id)"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-button type="success" @click="openAddColumnDialog">Add Column</el-button>
      <el-button type="danger" @click="confirmRemoveColumn">Remove Column</el-button>
      <el-button type="primary" @click="addRow">Add Row</el-button>
      <!-- Reorder Columns Button -->
      <el-button type="warning" @click="reorderDialogVisible = true">Reorder Columns</el-button>

      <!-- Add Column Dialog -->
      <el-dialog title="Add Column" :visible.sync="addColumnDialogVisible" @close="resetDialog">
        <el-form :model="newColumn" ref="addColumnForm">
          <el-form-item label="Column Name" :label-width="formLabelWidth">
            <el-input v-model="newColumn.name" placeholder="Enter column name"></el-input>
          </el-form-item>
          <el-form-item label="Data Type" :label-width="formLabelWidth">
            <el-select v-model="newColumn.type" placeholder="Select data type">
              <el-option label="String" value="string"></el-option>
              <el-option label="Number" value="number"></el-option>
              <el-option label="Boolean" value="boolean"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="Dummy Value" :label-width="formLabelWidth">
            <el-input v-model="newColumn.value" placeholder="Enter dummy value"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="addColumnDialogVisible = false">Cancel</el-button>
          <el-button type="primary" @click="addColumn">Add Column</el-button>
        </div>
      </el-dialog>

      <!-- Remove Column Confirmation Dialog -->
      <el-dialog title="Confirm Removal" :visible.sync="removeColumnDialogVisible">
        <el-form :model="columnToRemove">
          <el-form-item label="Select Column" :label-width="formLabelWidth">
            <el-select v-model="columnToRemove.name" placeholder="Select column to remove">
              <el-option v-for="column in columns" :key="column.name" :label="column.name" :value="column.name"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="removeColumnDialogVisible = false">Cancel</el-button>
          <el-button type="danger" @click="removeColumn">Remove Column</el-button>
        </div>
      </el-dialog>

      <!-- Reorder Columns Dialog -->
      <el-dialog title="Reorder Columns" :visible.sync="reorderDialogVisible">
        <el-form ref="reorderForm">
          <el-form-item label="Column Order" :label-width="formLabelWidth">
            <el-select v-model="newColumnOrder" placeholder="Select column order" multiple>
              <el-option v-for="column in columns" :key="column.name" :label="column.name" :value="column.name"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="reorderDialogVisible = false">Cancel</el-button>
          <el-button type="primary" @click="reorderColumns">Reorder Columns</el-button>
        </div>
      </el-dialog>
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
      rows: [],
      tableData: [],
      newColumn: { name: '', type: 'string', value: '' },
      addColumnDialogVisible: false,
      removeColumnDialogVisible: false,
      reorderDialogVisible: false,  // To control the reorder dialog visibility
      columnToRemove: { name: '' },
      newColumnOrder: [], // Store the new order of columns
      formLabelWidth: '120px',
      rowIdCounter: 0, // Counter to generate unique IDs for rows
    };
  },
  methods: {
    updateColumns() {
      const currentCount = this.columns.length;
      const targetCount = this.numColumns;

      // Add new columns if needed
      if (targetCount > currentCount) {
        for (let i = currentCount; i < targetCount; i++) {
          this.columns.push({ name: "", type: "string", value: "" });
        }
      } 
      // Remove extra columns if needed
      else if (targetCount < currentCount) {
        this.columns = this.columns.slice(0, targetCount);
      }
    },
    createTable() {
      this.step = 2;
      this.rows = [];
      this.tableData = [];

      // Initialize the table with rows and dummy values
      const newRow = { id: this.rowIdCounter++ };
      this.columns.forEach(column => {
        newRow[column.name] = column.value; // Use the dummy value for each column
      });
      this.rows.push(newRow);
      this.tableData = [...this.rows];
    },
    addRow() {
      // Create an empty row
      const newRow = { id: this.rowIdCounter++};
      this.columns.forEach(column => {
        newRow[column.name] = ""; // Empty values for new rows
      });
      this.rows.push(newRow);
      this.tableData.push(newRow);
    },
    removeRow(rowId) {
      this.tableData = this.tableData.filter(row => row.id !== rowId);
      this.rows = this.rows.filter(row => row.id !== rowId);
    },
    openAddColumnDialog() {
      this.addColumnDialogVisible = true;
    },
       addColumn() {
      if (this.newColumn.name && this.newColumn.type) {
        // Check if the column already exists
        const existingColumn = this.columns.find(column => column.name === this.newColumn.name);
        if (existingColumn) {
          this.$message.error('Column with this name already exists');
          return;
        }

        // Add the new column to the columns array
        this.columns.push({ ...this.newColumn });
        
        // Update existing rows to include the new column with its default value
        this.rows.forEach(row => {
          this.$set(row, this.newColumn.name, this.newColumn.value || "");
        });

        // Update table data to reflect the changes
        this.tableData = this.rows.map(row => {
          const newRow = {};
          this.columns.forEach(column => {
            newRow[column.name] = row[column.name] || "";
          });
          return newRow;
        });

        this.newColumn = { name: '', type: 'string', value: '' };
        this.addColumnDialogVisible = false;
      } else {
        this.$message.error('Please fill in the column details');
      }
    },
    confirmRemoveColumn() {
      this.removeColumnDialogVisible = true;
    },
    removeColumn() {
      const columnName = this.columnToRemove.name;
      this.columns = this.columns.filter(column => column.name !== columnName);

      // Remove the column from each row
      this.rows = this.rows.map(row => {
        const newRow = {};
        this.columns.forEach(column => {
          newRow[column.name] = row[column.name];
        });
        return newRow;
      });

      this.tableData = this.rows.map(row => {
        const newRow = {};
        this.columns.forEach(column => {
          newRow[column.name] = row[column.name];
        });
        return newRow;
      });

      this.columnToRemove = { name: '' };
      this.removeColumnDialogVisible = false;
    },
    reorderColumns() {
      if (this.newColumnOrder.length !== this.columns.length) {
        this.$message.error("Please select a valid order for all columns.");
        return;
      }

      // Create a new column array based on the selected order
      const reorderedColumns = this.newColumnOrder.map(name => {
        return this.columns.find(column => column.name === name);
      });

      // Reorder the tableData as well
      this.tableData = this.rows.map(row => {
        const newRow = {};
        reorderedColumns.forEach(column => {
          newRow[column.name] = row[column.name];
        });
        return newRow;
      });

      this.columns = reorderedColumns;
      this.reorderDialogVisible = false;
    },
    resetDialog() {
      this.newColumn = { name: '', type: 'string', value: '' };
    },
  },
};
</script>

<style scoped>
.column-form {
  margin-bottom: 10px;
}
</style>
