<template>
  <div>

    <LessonList @add-to-cart="addToCart" />
    <ShoppingCart :cart="cart" @remove-from-cart="removeFromCart" />
  </div>
</template>

<script>
import LessonList from './components/LessonList.vue';
import ShoppingCart from './components/ShoppingCart.vue';

export default {
  components: { LessonList, ShoppingCart },
  data() {
    return {
      cart: [],
      showCart: false
    };
  },
  methods: {
    addToCart(lesson) {
      this.cart.push(lesson);
    },
    removeFromCart(item) {
      const index = this.cart.findIndex(lesson => lesson._id === item._id);
      if (index !== -1) {
        this.cart.splice(index, 1);  // Remove only the first matching item
      }
    },
    togglePage() {
      this.showCart = !this.showCart; // Toggle the showCart flag
    },
  }
};
</script>
