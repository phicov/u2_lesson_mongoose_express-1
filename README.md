# ![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)  SOFTWARE ENGINEERING IMMERSIVE

## Getting started

1. Fork
1. Clone

# MongoDB, Mongoose, and Express: Building an API

## First! Database Design

So we are going to build an express api for products. 
The constraints are that a product has a title, a description, a price, brand name and a brand url.

How should we design our data model? Perhaps this way:

```js
const Product = new Schema(
    {
        title: { type: String, required: true },
        description: { type: String, required: true },
        price: { type: String, required: true },
        brand_name: { type: String, required: true },
        brand_url: { type: String, required: true }
    },
    { timestamps: true },
)
```

> Note: There is a more efficient way of modeling our data than the above. Let's explore that.

With the above design, if create products what we will notice quickly is that the brand name and brand url fields will repeat themselves, for example if we have 300 New Balance shoes in our database, we will repeat "New Balance" and "https://www.newbalance.com" 300 times! We can solve this by creating a brand model and have the product model refer to the brand model like this:

```js
const Product = new Schema(
    {
        title: { type: String, required: true },
        description: { type: String, required: true },
        price: { type: String, required: true },
        brand: { type: Schema.Types.ObjectId, ref: 'brands' }
    },
    { timestamps: true },
)
```

```js
const Brand = new Schema(
    {
        name: { type: String, required: true },
        url: { type: String, required: true }
    },
    { timestamps: true },
)
```

Now we create a brand only once (not 300 times) and have the product model reference it via the brand id. This is a more elegant approach.

Awesome! Now that we have our data model design 100% let's jump into coding this app!

Let's start!

```sh
cd mongodb-mongoose-express-api
npm init -y && npm install mongoose
mkdir db models seed
touch db/index.js models/{brand,product}.js seed/brandsProducts.js
```

Now let's open up Visual Studio Code and write some code:

```sh
code .
```

Inside our `db` folder we are going to use Mongoose to establish a connection to our MongoDB `productsDatabase`:

mongodb-mongoose-express-api/db/index.js
```js
const mongoose = require('mongoose')

mongoose
    .connect('mongodb://127.0.0.1:27017/productsDatabase', { useUnifiedTopology: true, useNewUrlParser: true })
    .then(() => {
        console.log('Successfully connected to MongoDB.');
      })
    .catch(e => {
        console.error('Connection error', e.message)
    })
// mongoose.set('debug', true)
const db = mongoose.connection

module.exports = db
```

Let's create our brand model:

mongodb-mongoose-express-api/models/brand.js
```js
const mongoose = require('mongoose')
const Schema = mongoose.Schema

const Brand = new Schema(
    {
        name: { type: String, required: true },
        url: { type: String, required: true }
    },
    { timestamps: true },
)

module.exports = mongoose.model('brands', Brand)
```

Now we can create our product model:

mongodb-mongoose-express-api/models/product.js
```js
const mongoose = require('mongoose')
const Schema = mongoose.Schema

const Product = new Schema(
    {
        title: { type: String, required: true },
        description: { type: String, required: true },
        price: { type: String, required: true },
        brand: { type: Schema.Types.ObjectId, ref: 'brands' }
    },
    { timestamps: true },
)

module.exports = mongoose.model('products', Product)
```

mongodb-mongoose-express-api/seed/brandsProducts.js
```js
const db = require('../db')
const Brand = require('../models/brand')
const Product = require('../models/product')

db.on('error', console.error.bind(console, 'MongoDB connection error:'))

const main = async () => {

    const brand1 = await new Brand({ name: 'Apple', url: 'https://www.apple.com' })
    brand1.save()

    const brand2 = await new Brand({ name: 'Vespa', url: 'https://www.vespa.com' })
    brand2.save()

    const brand3 = await new Brand({ name: 'New Balance', url: 'https://www.newbalance.com' })
    brand3.save()

    const brand4 = await new Brand({ name: 'Tribe', url: 'https://www.tribebicycles.com' })
    brand4.save()

    const brand5 = await new Brand({ name: 'Stumptown', url: 'https://www.stumptowncoffee.com' })
    brand5.save()

    const products = [
        { title: 'Apple AirPods', description: 'https://www.apple.com/airpods', price: '250', brand: brand1._id },
        { title: 'Apple iPhone Pro', description: 'https://www.apple.com/iphone-11-pro', price: '1000', brand: brand1._id },
        { title: 'Apple Watch', description: 'https://www.apple.com/watch', price: '499', brand: brand1._id },
        { title: 'Vespa Primavera', description: 'https://www.vespa.com/us_EN/vespa-models/primavera.html', price: '3000', brand: brand2._id },
        { title: 'New Balance 574 Core', description: 'https://www.newbalance.com/pd/574-core/ML574-EG.html', price: '84', brand: brand3._id },
        { title: 'Tribe Messenger Bike 004', description: 'https://tribebicycles.com/collections/messenger-series/products/mess-004-tx', price: '675', brand: brand4._id },
        { title: 'Stumptown Hair Bender Coffee', description: 'https://www.stumptowncoffee.com/products/hair-bender', price: '17', brand: brand5._id }
    ]

    await Product.insertMany(products)
    console.log("Created products!")
}

const run = async () => {
    await main()
    db.close()
}

run()
```

Awesome, so this products "seed" file above is a script that, once executed, connects to the Mongo database and creates 7 products in the products collection.

Let's execute our seed file:

```sh
node seed/brandsProducts.js
```

So how do we know if it worked? We could drop into the `mongo` interactive shell and check:

```sh
mongo
> use productsDatabase
> db.products.find()
```

Create a .gitignore file `touch .gitignore`!

```sh
/node_modules
.DS_Store
```

Cool, enough Mongoose. Now, Express. Let's install Express:

```sh
npm install express
```
And now let's create our Express boilerplate:

```sh
touch server.js
```

Add the code:

mongodb-mongoose-express-api/server.js
```js
const express = require('express');
const PORT = process.env.PORT || 3000;
const db = require("./db/index")

const app = express();

db.on('error', console.error.bind(console, 'MongoDB connection error:'))

app.get('/', (req, res) => {
  res.send("This is root!");
});

app.listen(PORT, () => {
  console.log(`Express server listening on port ${PORT}`);
});


```

Let's make sure our server works:

```sh
node server.js
open localhost:3000
```

Awesome! Next we want to be able to access our Product model from within the models folder.
Add the following to the top of your server.js file:

```js
const Product = require('./models/product');
```

Let's create the route to show all products:

```js
app.get('/products', async (req, res) => {
    const products = await Product.find()
    res.json(products)
})
```

Restart the server and test the route:

```sh
node server.js
```

Try it in your browser: http://localhost:3000/products

Now I would like to see a specific product.
Let's say you type http://localhost:3000/products/5e385d110909f66c6404fbc9 then our API should respond with the product where id equals 2. Express let's us do this via the `req.params` object:

```js
app.get('/products/:id', async (req, res) => {
  const { id } = req.params
  const product = await Product.findById(id)
  res.json(product)
})
```

What if the product does not exist in the database? We would get an ugly error message. We can handle this by using a try/catch block:

```js
app.get('/products/:id', async (req, res) => {
    try {
        const { id } = req.params
        const product = await Product.findById(id)
        if (!product) throw Error('Product not found')
        res.json(product)
    } catch (e) {
        console.log(e)
        res.send('Product not found!')
    }
})
```

Does it work? Restart the server and test the route.

```sh
node server.js
```

Open http://localhost:3000/products/5e385d110909f66c6404fbc9 in your browser.

Restarting the server can be a pain! Let's fix this by installing [nodemon](https://nodemon.io)!

```sh
npm install nodemon --save-dev
```

Modify your package.json file:

```js
....
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "nodemon server.js"
  },
....
```

Now run `npm start` and nodemon will watch for changes to your JavaScript files and restart your server :)

## Exercise

Create the following Express routes for Brands:

- http://localhost:3000/brands
- http://localhost:3000/brands/9e385d110909f66c6404ebs7

**Success!**

![](http://www.winsold.com/sites/all/modules/winsold/images/checkmark.svg)

## Feedback

> [Take a minute to give us feedback on this lesson so we can improve it!](https://forms.gle/vgUoXbzxPWf4oPCX6)
