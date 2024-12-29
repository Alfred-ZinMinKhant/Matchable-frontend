<template>
  <div class="bg-white shadow-lg rounded-lg p-6 mx-auto w-full">
    <h2 class="text-2xl font-bold text-blue-700 mb-6">Booking Form</h2>
    <form @submit.prevent="submitBooking">
      <div class="mb-4">
        <label for="name" class="block font-medium text-blue-600">Name</label>
        <input
          id="name"
          v-model="name"
          type="text"
          placeholder="Your Name"
          class="mt-2 block w-full p-2 border rounded"
          required
        />
      </div>
      <div class="mb-4">
        <label for="email" class="block font-medium text-blue-600">Email</label>
        <input
          id="email"
          v-model="email"
          type="email"
          placeholder="Your Email"
          class="mt-2 block w-full p-2 border rounded"
          required
        />
      </div>
      <div class="mb-4">
        <label for="phone" class="block font-medium text-blue-600">Phone</label>
        <input
          id="phone"
          v-model="phone"
          type="text"
          placeholder="Your Phone Number"
          class="mt-2 block w-full p-2 border rounded"
          required
        />
      </div>
      <div class="mb-6">
        <label class="flex items-center">
          <input v-model="terms" type="checkbox" class="mr-2" required />
          <span class="text-blue-600 font-medium">
            Agree to Terms & Conditions
          </span>
        </label>
      </div>
      <button
        class="bg-green-600 text-white font-medium py-2 px-6 rounded-lg hover:bg-green-700 transition-colors duration-300 w-full"
        type="submit"
      >
        Submit Booking
      </button>
    </form>
  </div>
</template>

<script>
export default {
  props: ["cart"],
  data() {
    return {
      name: "",
      email: "",
      phone: "",
      terms: false,
    };
  },
  methods: {
    async submitBooking() {
      if (this.cart.length === 0) {
        alert("Please add at least one session to your cart.");
        return;
      }

      const bookingData = {
        name: this.name,
        email: this.email,
        phone: this.phone,
        sessions: this.cart,
      };

      try {
        // Call backend to process booking and send email
        const response = await fetch(
          "https://matchable-backend.onrender.com/bookings",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(bookingData),
          }
        );

        if (response.ok) {
          alert(
            "Booking submitted successfully! Check your email for confirmation."
          );
          this.$emit("clear-cart"); // Clear cart after successful booking
          this.name = "";
          this.email = "";
          this.phone = "";
          this.terms = false;
        } else {
          alert("Failed to submit booking. Please try again.");
        }
      } catch (error) {
        console.error("Error submitting booking:", error);
        alert("An error occurred. Please try again.");
      }
    },
  },
};
</script>

<style scoped>
.bg-white {
  background-color: #ffffff;
}

.text-blue-700 {
  color: #1d4ed8;
}

.text-blue-600 {
  color: #2563eb;
}

.bg-green-600 {
  background-color: #16a34a;
}

.bg-green-700 {
  background-color: #15803d;
}
</style>
