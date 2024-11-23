<template>
  <div class="container mt-5">
    <div v-if="!showCart">
    <div>
      <h2 class="mt-5">Shopping Cart</h2>
      <ul class="list-group mb-4">
        <li v-for="item in cart" :key="item._id" class="list-group-item">
          <i class="fas fa-chalkboard-teacher"></i>
          {{ item.subject }} - ${{ item.price }} ({{ item.location }})
          <button class="btn btn-danger btn-sm float-right" @click="removeFromCart(item)">Remove</button>
        </li>
      </ul>
      <p v-if="cart.length === 0" class="text-muted">Your cart is empty.</p>

    </div>

    <h3>Checkout</h3>
    <div v-if="cart.length > 0">
      <div class="form-group">
        <label for="name">Name</label>
        <input type="text" id="name" class="form-control" v-model="name" placeholder="Enter your name">
      </div>
      <div class="form-group">
        <label for="phone">Phone</label>
        <input type="text" id="phone" class="form-control" v-model="phone" placeholder="Enter your phone number">
      </div>
      <button class="btn btn-success" :disabled="!isCheckoutValid" @click="checkout">Submit Order</button>
      <p v-if="checkoutMessage" class="alert alert-info mt-3">{{ checkoutMessage }}</p>
    </div>
  </div>
  </div>
</template>

<script>
export default {
  props: ["cart"],
  data() {
    return {
      name: "",
      phone: "",
      checkoutMessage: "",
    };
  },
  methods: {
    togglePage() {
      this.showCart = !this.showCart; // Toggle the showCart flag
    },
    checkout() {
      // Group items by lessonId and sum their quantities
      const groupedItems = [];
      const itemMap = new Map();

      this.cart.forEach((lesson) => {
        if (itemMap.has(lesson._id)) {
          // Increment quantity if lesson already exists
          itemMap.get(lesson._id).quantity += lesson.quantity || 1;
        } else {
          // Add new entry to the map
          itemMap.set(lesson._id, {
            lessonId: lesson._id,
            subject: lesson.subject,
            quantity: lesson.quantity || 1,
          });
        }
      });

      // Convert the map back to an array
      itemMap.forEach((value) => groupedItems.push(value));

      // Prepare the order
      const order = {
        name: this.name,
        phone: this.phone,
        items: groupedItems,
      };

      // Send the order to the server
      fetch("http://localhost:3000/collections/orders", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(order),
      })
          .then(() => {
            this.checkoutMessage = "Order placed successfully!";
            this.$emit("clear-cart"); // Emit event to clear the cart
          })
          .catch((error) => {
            console.error("Checkout error:", error);
            this.checkoutMessage = "Error placing the order. Please try again.";
          });
    },
    removeFromCart(item) {
      this.$emit('remove-from-cart', item);
      item.spaces++;
    },
  },
  computed: {
    isCheckoutValid() {
      const nameValid = /^[A-Za-z\s]+$/.test(this.name);
      const phoneValid = /^[0-9]+$/.test(this.phone);
      return nameValid && phoneValid && this.cart.length > 0;
    },
  },
};
</script>
