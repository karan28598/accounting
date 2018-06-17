<template>
    <div class="container">
        <div class="row">
            <div class="col-md-8 col-md-offset-2">
                <h1>Point of Sale</h1>
                <div class="row">
                    <div class="col-md-6">
                        <h6>Customer</h6>
                        <frappe-control
                            :docfield="customerDocfield"
                            :value="value"
                            @change="value => updateValue(this.customerDocfield.fieldname, value)"
                            :onlyInput="true"
                        />
                        <br>
                        <br>
                        <transaction :items="lineItems" :edit="toggleEdit" :remove="removeItem" ></transaction>
                        <div class="list-group">
                          <button class="list-group-item item" @click="createInvoice()">
                              <strong>Create Invoice</strong>
                          </button>
                        </div>
                        <br>
                        <div class="row">
                            <billing></billing>
                            <checkout></checkout>
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="row">
                            <div class="col-md-6">
                                <h6>Item Select</h6>
                                <frappe-control
                                    :docfield="itemDocfield"
                                    :value="i_value"
                                    @change="i_value => updateValue(this.itemDocfield.fieldname, i_value)"
                                    :onlyInput="true"
                                />
                            </div>
                        </div>
                        <br>
                        <item-list :items="items" :add="onItemClick"></item-list>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import Transaction from "./Transaction";
import Billing from "./Billing";
import Checkout from "./Checkout";
import ItemList from "./ItemList";
import frappe from "frappejs";
import FrappeControl from './controls/FrappeControl';

export default {
  components: {
    Transaction,
    ItemList,
    Billing,
    Checkout
  },
  data() {
    this.customerDocfield={
      fieldname: "customer",
      label: "Customer",
      fieldtype: "Link",
      target: "Party"
    };
    this.itemDocfield={
      fieldname: "name",
      label: "Item Name",
      fieldtype: "Link",
      target: "Item"
    };
    return {
      items: [],
      lineItems: []
    };
  },
  async created(){ 
  this.items=await frappe.db.getAll({
          doctype: "Item",
          fields: ["name", "rate"],
        });
      this.grandTotal=0;
      this.netTotal=0;
  /*this.items=it.filter(function(el) {
      return el.name.toLowerCase().indexOf(this.itemfilter.toLowerCase()) > -1;
      })*/
  },
  methods: {
    onItemClick: function(item) {
      console.log("in", item);
      var found = false;

      for (var i = 0; i < this.lineItems.length; i++) {
        if (this.lineItems[i].item === item) {
          this.lineItems[i].numberOfItems++;
          found = true;
          break;
        }
      }
  
      if (!found) {
        this.lineItems.push({ item: item, numberOfItems: 1, editing: false });
      }
      this.tempInvoice();
    },
    toggleEdit: function(lineItem) {
      lineItem.editing = !lineItem.editing;
    },
    removeItem: function(lineItem) {
      for (var i = 0; i < this.lineItems.length; i++) {
        if (this.lineItems[i] === lineItem) {
          this.lineItems.splice(i, 1);
          break;
        }
      }
      this.tempInvoice();
    },
    updateValue(field, value) {
                  this.value = value;
                },
    async tempInvoice(){
      var temp_item=[];
      for(var i=0;i<this.lineItems.length;i++)
      {
        console.log(this.lineItems[i].item.name+" "+this.lineItems[i].numberOfItems);
        var temp={
          item:this.lineItems[i].item.name,
          quantity:this.lineItems[i].numberOfItems
        };
        temp_item.push(temp);
      }
      let tempdoc = await frappe.newDoc({
        doctype: 'Invoice', 
        name: 'something',
        customer:'Test Customer',
        items:temp_item
        });
      await tempdoc.applyChange()
      this.grandTotal=tempdoc.grandTotal;
      this.netTotal=tempdoc.netTotal;
    },
    async createInvoice(){
      if(!this.lineItems.length)
        alert("No items selcted");
      else{
      var final_item=[];
      for(var i=0;i<this.lineItems.length;i++)
      {
        console.log(this.lineItems[i].item.name+" "+this.lineItems[i].numberOfItems);
        var temp={
          item:this.lineItems[i].item.name,
          quantity:this.lineItems[i].numberOfItems
        };
        final_item.push(temp);
      }
      frappe.insert({
            doctype:'Invoice',
            customer: this.value,
            items:final_item
        });
      alert("Invoice added");
        }
    }
  }
};
</script>
<style>
</style>
