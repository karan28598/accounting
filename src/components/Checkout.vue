<template>
<div class="col-md-6">
  <div class="list-group">
        <button class="list-group-item item" @click="checkout()">
            <strong>Make Payment</strong>
        </button>
    </div>
</div>
</template>

<script>
export default {
  methods: {
    checkout() {
      // this.$checkout.close() 
      // is also available.
      this.$checkout.open({
        name: 'Shut up and take my money!',
        currency: 'INR',
        amount: 10000,
        token: (token) => {
          // Send the token to your server
          // for payment or subscription handling,
          // or do whatever you want with it
          // I don't really care. 
        } 
      });
    }
  }
}
</script>