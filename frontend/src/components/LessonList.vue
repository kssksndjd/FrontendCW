<template>
  <!-- Add Bootstrap CDN for basic styling -->
  <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
  <!-- Add Font Awesome CDN for icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">

  <div class="container mt-5">
    <h1 class="text-center mb-4">After School Classes!!</h1>

    <!-- Toggle View Button -->
    <button class="btn btn-info mb-4" @click="togglePage">
      {{ showCart ? 'Back to Lessons' : 'View Shopping Cart' }}
    </button>

    <div v-if="!showCart">
      <!-- Search Input -->
      <div class="form-group">
        <label for="searchQuery">Search Lessons:</label>
        <input
            type="text"
            id="searchQuery"
            v-model="searchQuery"
            class="form-control"
            placeholder="Search by subject, location, price, or spaces"
        />
      </div>

      <!-- Sort Options -->
      <div class="form-group">
        <label for="sortKey">Sort By:</label>
        <select v-model="sortKey" id="sortKey" class="form-control">
          <option value="subject">Subject</option>
          <option value="location">Location</option>
          <option value="price">Price</option>
          <option value="spaces">Spaces</option>
        </select>
        <!-- Sort Order Toggle -->
        <button class="btn btn-secondary mt-2" @click="toggleSortOrder">
          Sort: {{ sortOrder === 'asc' ? 'Ascending' : 'Descending' }}
        </button>
      </div>

      <!-- Display Lessons -->
      <div class="row">
        <div class="col-md-4" v-for="lesson in filteredAndSortedLessons" :key="lesson._id">
          <div class="card mb-4">
            <img :src="lesson.image" class="card-img-top lesson-image" alt="Lesson Image" />
            <div class="card-body">
              <h5 class="card-title">
                <i class="fas fa-chalkboard-teacher"></i> <!-- Font Awesome Icon -->
                {{ lesson.subject }}
              </h5>
              <p class="card-text">Location: {{ lesson.location }}</p>
              <p class="card-text">Price: ${{ lesson.price }}</p>
              <p class="card-text">Spaces Available: {{ lesson.spaces }}</p>
              <button class="btn btn-primary" :disabled="lesson.spaces === 0" @click="addToCart(lesson)">
                {{ lesson.spaces === 0 ? 'Sold Out' : 'Add to Cart' }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchQuery: '', // Search input for filtering lessons
      sortKey: 'subject', // Key to sort by
      sortOrder: 'asc', // Sorting order (ascending by default)
      showCart: false, // Toggle between cart and lessons page
      lessons: [], // Lesson data from the API
    };
  },
  methods: {
    async fetchLessons() {
      const response = await fetch('http://localhost:3000/collections/lessons'); // Ensure endpoint is correct
      this.lessons = await response.json();
    },
    addToCart(lesson) {
      if (lesson.spaces > 0) {
        this.$emit('add-to-cart', lesson);
        lesson.spaces--;
      }
    },
    togglePage() {
      this.showCart = !this.showCart; // Toggle the showCart flag
    },
    toggleSortOrder() {
      this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
    },
  },
  computed: {
    // Filter and sort lessons
    filteredAndSortedLessons() {
      // Filter lessons based on search query
      let filtered = this.lessons.filter((lesson) => {
        const query = this.searchQuery.toLowerCase();
        return (
            lesson.subject.toLowerCase().includes(query) ||
            lesson.location.toLowerCase().includes(query) ||
            lesson.price.toString().includes(query) ||
            lesson.spaces.toString().includes(query)
        );
      });

      // Sort lessons based on sortKey and sortOrder
      return filtered.sort((a, b) => {
        let result;
        if (this.sortKey === 'price' || this.sortKey === 'spaces') {
          result = a[this.sortKey] - b[this.sortKey];
        } else {
          result = a[this.sortKey].localeCompare(b[this.sortKey]);
        }
        return this.sortOrder === 'asc' ? result : -result;
      });
    },
  },
  mounted() {
    this.fetchLessons();
  },
};
</script>

<style>
.lesson-image {
  width: 100%; /* Make the image span the card's width */
  height: 200px; /* Fixed height for consistency */
  object-fit: cover; /* Ensure the image fills the space without distortion */
  border-radius: 5px; /* Optional: Rounded corners for aesthetics */
}
</style>
