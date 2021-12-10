# Mongoose and Express

![MongooseExpress](https://i.imgur.com/nDoaxGh.png)

## Getting started

1. Fork
1. Clone

# MongoDB, Mongoose, and Express: Building an API

## Database Design

We are going to build an express api for products.
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
  { timestamps: true }
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
  { timestamps: true }
)
```

```js
const Brand = new Schema(
  {
    name: { type: String, required: true },
    url: { type: String, required: true }
  },
  { timestamps: true }
)
```

Now we create a brand only once (not 300 times) and have the product model reference it via the brand id. This is a more elegant approach.

Awesome! Now that we have our data model design 100% let's jump into coding this app!

Let's start!

```sh
npm init -y
npm install mongoose
mkdir db models seed
touch db/index.js models/{brand,product,index}.js seed/brandsProducts.js
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
  .connect('mongodb://127.0.0.1:27017/productsDatabase')
  .then(() => {
    console.log('Successfully connected to MongoDB.')
  })
  .catch((e) => {
    console.error('Connection error', e.message)
  })
// mongoose.set('debug', true)
const db = mongoose.connection

module.exports = db
```

Let's create our brand schema:

mongodb-mongoose-express-api/models/brand.js

```js
const { Schema } = require('mongoose')

const Brand = new Schema(
  {
    name: { type: String, required: true },
    url: { type: String, required: true }
  },
  { timestamps: true }
)

module.exports = Brand
```

Now we can create our product schema:

mongodb-mongoose-express-api/models/product.js

```js
const { Schema } = require('mongoose')

const Product = new Schema(
  {
    title: { type: String, required: true },
    description: { type: String, required: true },
    price: { type: String, required: true },
    brand: { type: Schema.Types.ObjectId, ref: 'brands' }
  },
  { timestamps: true }
)

module.exports = Product
```

Next we'll set up our models:

`models/index.js`

```js
const mongoose = require('mongoose')
const ProductSchema = require('./Product')
const BrandSchema = require('./Brand')

const Product = mongoose.model('products', ProductSchema)
const Brand = mongoose.model('brands', BrandSchema)

module.exports = {
  Product,
  Brand
}
```

`seed/brandsProducts.js`

```js
const db = require('../db')
const { Brand, Product } = require('../models')

db.on('error', console.error.bind(console, 'MongoDB connection error:'))

const main = async () => {
  const brand1 = await new Brand({
    name: 'Apple',
    url: 'https://www.apple.com'
  })
  brand1.save()

  const brand2 = await new Brand({
    name: 'Vespa',
    url: 'https://www.vespa.com'
  })
  brand2.save()

  const brand3 = await new Brand({
    name: 'New Balance',
    url: 'https://www.newbalance.com'
  })
  brand3.save()

  const brand4 = await new Brand({
    name: 'Tribe',
    url: 'https://www.tribebicycles.com'
  })
  brand4.save()

  const brand5 = await new Brand({
    name: 'Stumptown',
    url: 'https://www.stumptowncoffee.com'
  })
  brand5.save()

  const products = [
    {
      title: 'Apple AirPods',
      description: 'https://www.apple.com/airpods',
      price: '250',
      brand: brand1._id
    },
    {
      title: 'Apple iPhone Pro',
      description: 'https://www.apple.com/iphone-11-pro',
      price: '1000',
      brand: brand1._id
    },
    {
      title: 'Apple Watch',
      description: 'https://www.apple.com/watch',
      price: '499',
      brand: brand1._id
    },
    {
      title: 'Vespa Primavera',
      description: 'https://www.vespa.com/us_EN/vespa-models/primavera.html',
      price: '3000',
      brand: brand2._id
    },
    {
      title: 'New Balance 574 Core',
      description: 'https://www.newbalance.com/pd/574-core/ML574-EG.html',
      price: '84',
      brand: brand3._id
    },
    {
      title: 'Tribe Messenger Bike 004',
      description:
        'https://tribebicycles.com/collections/messenger-series/products/mess-004-tx',
      price: '675',
      brand: brand4._id
    },
    {
      title: 'Stumptown Hair Bender Coffee',
      description: 'https://www.stumptowncoffee.com/products/hair-bender',
      price: '17',
      brand: brand5._id
    }
  ]

  await Product.insertMany(products)
  console.log('Created products!')
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

So how do we know if it worked? We could drop into the `mongosh` interactive shell and check:

```sh
mongosh
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
npm install express cors morgan
npm install nodemon --dev
```

And now let's create our Express boilerplate:

```sh
touch server.js
```

Add the code:

`server.js`

```js
const express = require('express')
const cors = require('cors')
const logger = require('morgan')
const PORT = process.env.PORT || 3001
const db = require('./db')

const app = express()

app.use(cors())
app.use(express.json())
app.use(logger('dev'))

app.get('/', (req, res) => {
  res.send('This is root!')
})

app.listen(PORT, () => {
  console.log(`Express server listening on port ${PORT}`)
})
```

Let's make sure our server works, add the following scripts to your `package.json`:

```sh
"scripts": {
    "start":"node server.js",
    "dev":"nodemon server.js"
}
```

Run:

```sh
npm run dev
```

Awesome! Next we want to be able to access our Product model from within the models folder.
Add the following to the top of your server.js file:

```js
const { Product } = require('./models')
```

Let's create the route to show all products:

```js
app.get('/products', async (req, res) => {
  const products = await Product.find()
  res.json(products)
})
```

Test the route using `insomnia`:

```http
[GET] http://localhost:3001/products
```

Now I would like to see a specific product. Express let's us do this via the `req.params` object:

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

Test http://localhost:3001/products/:product_id in insomnia.

## Exercise

Create the following Express routes for Brands:

- http://localhost:3001/brands
- http://localhost:3001/brands/:brand_id

**Success!**

![](http://www.winsold.com/sites/all/modules/winsold/images/checkmark.svg)
