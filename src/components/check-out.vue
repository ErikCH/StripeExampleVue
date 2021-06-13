<template>
  <section class="showcase">
    <div class="nes-container with-title">
      <h3>Donkey Kong Country SNES $19.99</h3>
      <div class="img">
        <img
          src="https://stripe-camo.global.ssl.fastly.net/6d097cce2c90965a3c955721cead3e8e0a9f8050f4122b2b3c50762fbffb326a/68747470733a2f2f66696c65732e7374726970652e636f6d2f6c696e6b732f4d44423864584e6c636c38775155526f654464315454683556566c4a53327846544642554e48786d624639305a584e3058316c7963464a58546e52726556424863336b35626c5648596b5a554e45706a656730305a7a5a565335315a"
          alt=""
          srcset=""
        />
      </div>
    </div>
  </section>
  <div class="nes-container with-title is-centered">
    <form @submit.prevent="handleSubmit">
      <fieldset :class="{ dis: loading }" class="fields">
        <div class="nes-field"></div>
        <div class="nes-field">
          <label for="name_field">Name</label>
          <input
            placeholder="Jane Doe"
            type="text"
            name="name"
            id="name_field"
            class="nes-input"
          />
        </div>
        <div class="nes-field">
          <label for="email_field">Email</label>
          <input
            placeholder="jane.doe@example.com "
            type="email"
            name="email"
            id="email_field"
            class="nes-input"
          />
        </div>
        <div class="nes-field">
          <label for="address_field">Address</label>
          <input
            placeholder="1234 Sycamore Street"
            type="text"
            name="address"
            id="address_field"
            class="nes-input"
          />
        </div>
        <div class="nes-field">
          <label for="city_field">City</label>
          <input
            placeholder="Reno"
            type="text"
            name="city"
            id="city_field"
            class="nes-input"
          />
        </div>
        <div class="nes-field">
          <label for="state_field">State</label>
          <input
            placeholder="Nevada"
            type="text"
            name="state"
            id="state_field"
            class="nes-input"
          />
        </div>
        <div class="nes-field">
          <label for="zip_field">Zip</label>
          <input
            placeholder="89523"
            type="text"
            name="zip"
            id="zip_field"
            class="nes-input"
          />
        </div>
        <div class="nes-field">
          <label for="email_field">Credit Card</label>
          <div id="stripe-element-mount-point" class="nes-input" />
        </div>
      </fieldset>
      <div class="nes-field">
        <button
          type="submit"
          class="nes-btn is-primary"
          :class="{ dis: loading }"
        >
          {{ loading ? "Loading..." : "Pay $19.99" }}
        </button>
      </div>
    </form>
  </div>
  <h2>Or...</h2>
  <div class="nes-field mt">
    <button type="button" class="nes-btn is-success" @click="redirect">
      Pay $19.99 Page
    </button>
  </div>
</template>

<script>
import { loadStripe } from "@stripe/stripe-js";
const style = {
  style: {
    base: {
      iconColor: "#000",
      color: "#000",
      fontWeight: "800",
      fontFamily: "Press Start 2P",
      fontSize: "22px",
      fontSmoothing: "antialiased",
      ":-webkit-autofill": {
        color: "#fce883"
      },
      "::placeholder": {
        color: "green"
      }
    },
    invalid: {
      iconColor: "#FFC7EE",
      color: "red"
    }
  }
};
import { onMounted, ref } from "vue";
import { useRouter, useRoute } from "vue-router";

export default {
  setup() {
    const router = useRouter();
    let stripe = null;
    let loading = ref(true);
    let elements = null;
    onMounted(async () => {
      const ELEMENT_TYPE = "card";

      stripe = await loadStripe(import.meta.env.VITE_STRIPE_KEY);

      elements = stripe.elements();
      const element = elements.create(ELEMENT_TYPE, style);
      element.mount("#stripe-element-mount-point");
      loading.value = false;
    });

    async function handleSubmit(event) {
      if (loading.value) return;
      loading.value = true;
      const { name, email, address, city, state, zip } = Object.fromEntries(
        new FormData(event.target)
      );
      console.log("here", name, email, address, city, state, zip);

      const billingDetails = {
        name,
        email,
        address: {
          city,
          line1: address,
          state,
          postal_code: zip
        }
      };

      const cardElement = elements.getElement("card");

      try {
        const response = await fetch("http://localhost:3001/stripe", {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify({ amount: 1999 })
        });
        const { secret } = await response.json();
        console.log("secret", secret);

        const paymentMethodReq = await stripe.createPaymentMethod({
          type: "card",
          card: cardElement,
          billing_details: billingDetails
        });

        console.log("error?", paymentMethodReq);

        const { error } = await stripe.confirmCardPayment(secret, {
          payment_method: paymentMethodReq.paymentMethod.id
        });
        loading.value = false;
        console.log("error?", error);
        router.push("/success");
      } catch (error) {
        console.log("error", error);
        loading.value = false;
      }
    }

    function redirect() {
      stripe.redirectToCheckout({
        successUrl: "http://localhost:3000/success",
        cancelUrl: "http://localhost:3000",
        lineItems: [
          {
            price: "price_0J1wDR0ADhx7uM8yPL8Wmpoq",
            quantity: 1
          }
        ],
        mode: "payment"
      });
    }
    return { redirect, loading, handleSubmit };
  }
};
</script>

<style scoped>
.checkout {
  border: 1px solid black;
  padding: 3px;
}

.fields {
  display: flex;
  flex-wrap: wrap;
  flex-direction: column;
  gap: 30px;
}
.img {
  display: flex;
  justify-content: center;
}
.showcase {
  margin-bottom: 20px;
}
.mt {
  margin-top: 20px;
}
.dis {
  opacity: 0.5;
  cursor: default;
  pointer-events: none;
}
button {
  margin-top: 30px;
}
</style>
