<template>
  <div id="app" class="container">
    <form class="form">
      <h1>Create Invoice</h1>
      <div class="form-detail">
        <label>Customer Name</label>
        <input
          class="input-ele"
          type="text"
          v-model="customerName"
          required="true"
          placeholder="Name"
        />
      </div>
      <div class="form-detail">
        <label>Customer Email</label>
        <input
          class="input-ele"
          type="text"
          required="true"
          placeholder="Email"
          v-model="customerEmail"
        />
      </div>
      <ag-grid-vue
        style="width: 90%; height: 300px"
        class="ag-theme-alpine"
        :columnDefs="columnDefs"
        :rowData="rowData"
        rowSelection="multiple"
        :suppressRowClickSelection="true"
        :singleClickEdit="true"
        @grid-ready="onGridReady"
        :gridOptiopns="gridOptions"
      >
      </ag-grid-vue>

      <div class="form-group" style="align-self: flex-end; margin-right: 100px">
        <button @click="addNewRow()" type="button" class="btn">Add Row</button>
        <button class="btn" @click="deleteSelected()" type="button">
          Delete Selected Rows
        </button>
      </div>

      <hr />

      <div class="form-group">
        <button @click="submit()" type="submit" class="btn">
          Create Invoice
        </button>
      </div>
    </form>
  </div>
</template>

<script>
import { AgGridVue } from "ag-grid-vue";
import Stripe from "stripe";
const stripe = new Stripe(
  "sk_test_51LM6KQSHdSbMCUAPUzbEWLfXxEJYQPXNGkDzUZmZYs3TfeWsjIBRXjhjCsULHwlliJDPJfc4C9MgdltGtRP5p3s900YqZdUgS5"
);

export default {
  name: "App",
  data() {
    return {
      customerName: null,
      customerEmail: null,
      columnDefs: null,
      rowData: null,
      gridApi: null,
      columnApi: null,
      gridOptions: {
        defaultColDef: {
          sortable: true,
          resizable: true,
          filter: true,
        },
      },
    };
  },

  components: {
    AgGridVue,
  },

  beforeMount() {
    this.columnDefs = [
      {
        checkboxSelection: true,
        headerName: "ITEM NAME",
        field: "itemName",
        editable: true,
      },
      { headerName: "QTY", field: "qty", editable: true },
      { headerName: "Price(INR)", field: "price", editable: true },
    ];

    this.rowData = [
      { itemName: "Dinning Table", qty: "1", price: "30000" },
      { itemName: "Chair", qty: "4", price: "20000" },
    ];
  },

  methods: {
    onGridReady(params) {
      this.gridApi = params.api;
      this.columnApi = params.columnApi;
      params.api.sizeColumnsToFit();
    },

    addNewRow() {
      const emptyrow = {
        itemName: "",
        qty: "",
        price: "",
      };
      this.rowData.push(emptyrow);
      this.gridApi.applyTransaction({ add: [emptyrow] });
    },

    deleteSelected() {
      const selectedNodes = this.gridApi.getSelectedNodes();
      const selectedData = selectedNodes.map((node) => node.data);
      let rem = [];
      this.rowData.forEach((x) => {
        let g = "0";
        selectedData.forEach((y) => {
          if (x.itemName === y.itemName) {
            g = "1";
          }
        });
        if (g == "0") {
          rem.push(x);
        }
      });
      this.rowData = rem;
      this.gridApi.applyTransaction({ remove: selectedData });
    },

    async submit() {
      let p = "0";
      if (this.rowData.length < 1) {
        alert("Add atleast one item to the table");
        return;
      }
      this.rowData.forEach((x) => {
        if (
          x.itemName.length === 0 &&
          x.qty.length === 0 &&
          x.price.length === 0
        ) {
          p = "1";
        }
      });
      if (p == "1") {
        alert("No empty rows allowed");
        return;
      }

      const customer = await stripe.customers.create({
        email: "customer@example.com",
      });

      console.log(customer.id);
    },
  },
};
</script>

<style lang="scss">
@import "~ag-grid-community/styles/ag-grid.css";
@import "~ag-grid-community/styles/ag-theme-alpine.css";
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
.form {
  width: 70%;
  height: 80%;
  display: flex;
  flex-direction: column;
  text-align: center;
}
.form-detail {
  display: flex;
  justify-content: space-around;
  margin: 10px;
}

.input-ele {
  height: 20px;
  width: 300px;
}

.btn {
  height: 25px;
  margin: 5px;
}
</style>
