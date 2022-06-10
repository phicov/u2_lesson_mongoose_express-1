# Mongoose and Express

![MongooseExpress](https://i.imgur.com/nDoaxGh.png)

## Getting started

1. Fork
1. Clone

# MongoDB, Mongoose, and Express: Building an API

## Database Design

We are going to build an Express API for `products`.
A `product` will consist of the following attributes:
- `title`
- `description`
- `price` 
- `brandName`
- `brandUrl`.

How should we design our data model? Perhaps this way:

```js
const productSchema = new Schema(
  {
    title: { type: String, required: true },
    description: { type: String, required: true },
    price: { type: String, required: true },
    brandName: { type: String, required: true },
    brandUrl: { type: String, required: true }
  },
  { timestamps: true }
)
```

> Note: There is a more efficient way of modeling our data than the above. Let's explore that.

With the above design, as we create more products the opportunity for needless duplication is increased. For example, if we have 300 instances of "New Balance" shoes in our database, there will also be 300 instances of the values "New Balance" and "https://www.newbalance.com"! We can solve this by creating a separate `Brand` model and have the `Product` model reference it like so:

```js
const brandSchema = new Schema(
  {
    name: { type: String, required: true },
    url: { type: String, required: true }
  },
  { timestamps: true }
)
```

```js
const productSchema = new Schema(
  {
    title: { type: String, required: true },
    description: { type: String, required: true },
    price: { type: String, required: true },
    brand: { type: Schema.Types.ObjectId, ref: 'Brand' } // this line references the newly created 'Brand' model.
  },
  { timestamps: true }
)
```

By creating an instance of a `Brand` and simply referencing that instance in the `Product`, we help to reduce excessive data duplication and minize the opportunity for creating bugs (ex: a misspelling of "Nw Balaance" in several instances of a `Product` would be problematic if one were to search for all instances of "New Balance".)

Let's build our server and connect it to our _local_ Mongo database.

In the terminal:
```sh
npm init -y
npm install mongoose
mkdir db models seed
touch db/index.js models/{brand,product,index}.js seed/brandsProducts.js
```

Open the project with VSCode:
```sh
code .
```

Inside our `db` folder we will configure Mongoose to establish a connection to our local MongoDB `productsDatabase`:

mongodb-mongoose-express-api/db/index.js

```js
const mongoose = require('mongoose')

mongoose
  .connect('mongodb://127.0.0.1:27017/productsDatabase') // if we don't have a local database named "productsDatabase" one will be created upon a successful connection
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

Let's create our `Brand` schema:

mongodb-mongoose-express-api/models/brand.js

```js
const { Schema } = require('mongoose')

const brandSchema = new Schema(
  {
    name: { type: String, required: true },
    url: { type: String, required: true }
  },
  { timestamps: true }
)

module.exports = brandSchema
```

Let's also create our `Product` schema:

mongodb-mongoose-express-api/models/product.js

```js
const { Schema } = require('mongoose')

const productSchema = new Schema(
  {
    title: { type: String, required: true },
    description: { type: String, required: true },
    price: { type: String, required: true },
    brand: { type: Schema.Types.ObjectId, ref: 'Brand' }
  },
  { timestamps: true }
)

module.exports = productSchema
```

Next, we'll set up our models:

`models/index.js`

```js
const mongoose = require('mongoose')
const productSchema = require('./Product')
const brandSchema = require('./Brand')

const Product = mongoose.model('Products', productSchema)
const Brand = mongoose.model('Brands', brandSchema)

module.exports = {
  Product,
  Brand
}
```

With our schemas now defined we can create a "seed" file that, when run, will quickly populate our local database with instances of the `Brand` model and the `Product` model.

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

Let's execute our seed file in the terminal:
```sh
node seed/brandsProducts.js
```

To verify that the seed file created our data we can run `mongosh` interactive shell and check:

```sh
mongosh
> use productsDatabase
> db.products.find({})
```

Once we have verified our data was created, create a `.gitignore` in the root of this directory and add the following:

```sh
/node_modules
.DS_Store
```
Adding `node_modules` to our `.gitignore` file (before making a git commit) ensures that we will not track those files.

From here we can install the dependencies we want to build an Express server incorporating Mongoose:

```sh
npm install express cors morgan
npm install nodemon --include=dev
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
Add the following above your routes in your server.js file:

```js
const { Product } = require('./models')
```

Let's create the route to show all products:
```js
// server.js
app.get('/products', async (req, res) => {
  const products = await Product.find({})
  res.json(products)
})
```

Test the route using `insomnia`:

```http
[GET] http://localhost:3001/products
```

Let's create a route to show a specific product. 
`req.params`, provided by Express, is helpful in this scenario:

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

Test http://localhost:3001/products/:id in insomnia.

## Exercise

Create the following Express routes for Brands:

- http://localhost:3001/brands
- http://localhost:3001/brands/:id

**Success!**

![](http://www.winsold.com/sites/all/modules/winsold/images/checkmark.svg)
