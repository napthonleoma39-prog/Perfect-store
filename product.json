require('dotenv').config();
const express = require('express');
const app = express();
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const cors = require('cors');
const PRODUCTS = require('./products.json');
const PORT = process.env.PORT || 4242;
const CLIENT_URL = process.env.CLIENT_URL || 'https://example.com';

app.use(cors());
app.use(express.static('public'));
app.use(express.json());

// serve products to frontend
app.get('/products', (req, res) => {
  res.json(PRODUCTS);
});

// create a Stripe Checkout Session
app.post('/create-checkout-session', async (req, res) => {
  try {
    const { items, email } = req.body; // items: [{id, quantity}]
    if (!items || !Array.isArray(items) || items.length === 0) {
      return res.status(400).json({ error: 'No items' });
    }

    // Build line items
    const line_items = items.map(i => {
      const prod = PRODUCTS.find(p => p.id === i.id);
      if (!prod) throw new Error('Product not found: ' + i.id);
      return {
        price_data: {
          currency: 'usd',
          product_data: { name: prod.name, images: [prod.image] },
          unit_amount: Math.round(prod.price * 100)
        },
        quantity: i.quantity
      };
    });

    const session = await stripe.checkout.sessions.create({
      payment_method_types: ['card'],
      line_items,
      mode: 'payment',
      customer_email: email || undefined,
      success_url: `${CLIENT_URL}/success.html?session_id={CHECKOUT_SESSION_ID}`,
      cancel_url: `${CLIENT_URL}/cancel.html`
    });

    res.json({ url: session.url });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: err.message });
  }
});

app.listen(PORT, () => console.log('Server running on port', PORT));
