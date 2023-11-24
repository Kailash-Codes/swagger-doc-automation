
# Automated api documentation using swagger for node js.

This is all about how to integrate swagger for documenting nodejs api.
If you follow these steps properly, you will be able to document the nodejs api in an automated manner.

# Steps

1. initialize node js project
```
npm init -y or yarn init -y
```
2. Add required packages
```
npm install express && npm install -D nodemon 
```
3. Install required packages for swagger api documentation
```
npm install swagger-autogen swagger-ui-express mongoose-to-swagger
```
4. Create a file name swagger.js and insert initial swagger config
```
const { userSwagger } = require(".");

const swaggerAutogen = require("swagger-autogen")();

const doc = {
  info: {
    version: "", // by default: '1.0.0'
    title: "", // by default: 'REST API'
    description: "", // by default: ''
  },
  host: "", // by default: 'localhost:3000'
  basePath: "", // by default: '/'
  schemes: [], // by default: ['http']
  consumes: [], // by default: ['application/json']
  produces: [], // by default: ['application/json']
  tags: [
    // by default: empty Array
    {
      name: "", // Tag name
      description: "", // Tag description
    },
    // { ... }
  ],
  securityDefinitions: {
    bearerAuth: {
      type: "bearer",
      scheme: "bearer",
    },
  }, // by default: empty object
  definitions: {}, // by default: empty object
  
};

const outputFile = "./swagger-output.json";
const routes = ["./index.js"]; //if you are using express

/* NOTE: If you are using the express Router, you must pass in the 'routes' only the 
root file where the route starts, such as index.js, app.js, routes.js, etc ... */

swaggerAutogen(outputFile, routes, doc);
```

5. Add a script in package.json that will run swagger.json and make a new file named swagger-output.json
```
"scripts": {
    "start": "nodemon index.js",
    "swagger": "node ./swagger.js"
  },
  ```

6. This will work for all the get requests but now we are automating the post request which will add our fields of mongoose into form or body.

7. Add swagger schema to your model using mongoose-to-swagger package which we had installed earlier
E.g
if your model is Book and you created it as 
```
const m2s = require("mongoose-to-swagger);
const mongoose = require("mongoose);
const bookSchema = new mongoose.Schema({
    name :{
        type:String,
        required : true
    }
    author:{
        type:String,
        required : true
    }
    publishedDate :{
        type:Date,
        required : true
    }
})
const bookModel = mongoose.model("Book",bookSchema);

//add this 
const bookSchemaSwagger = {type:"object",...m2s(bookModel)};

module.exports = bookModel;
module.exports = {bookSchemaSwagger}
```

8. Now update you swagger.js to append this schema. Add this object
```
//import bookSchemaSwagger
components: {
    schemas: {
      someSchema: bookSchemaSwagger,
    },
  },
  ```
your swagger.js may look like this
```
const { bookSchemaSwagger } = require(".");

const swaggerAutogen = require("swagger-autogen")();

const doc = {
  info: {
    version: "", // by default: '1.0.0'
    title: "", // by default: 'REST API'
    description: "", // by default: ''
  },
  host: "", // by default: 'localhost:3000'
  basePath: "", // by default: '/'
  schemes: [], // by default: ['http']
  consumes: [], // by default: ['application/json']
  produces: [], // by default: ['application/json']
  tags: [
    // by default: empty Array
    {
      name: "", // Tag name
      description: "", // Tag description
    },
    // { ... }
  ],
  securityDefinitions: {
    bearerAuth: {
      type: "bearer",
      scheme: "bearer",
    },
  }, // by default: empty object
  definitions: {}, // by default: empty object
  components: {
    schemas: {
      someSchema: bookSchemaSwagger,
    },
  },
};

const outputFile = "./swagger-output.json";
const routes = ["./index.js"];

/* NOTE: If you are using the express Router, you must pass in the 'routes' only the 
root file where the route starts, such as index.js, app.js, routes.js, etc ... */

swaggerAutogen(outputFile, routes, doc);

```

9. We are now ready to go. Final thing, add this annotation in every post route you have.
```
 /*    #swagger.parameters['body'] = {
            in: 'body',
            description: 'User data.',
            required: true,
            schema: {
$ref: "#/components/schemas/someSchema"
            }
        }
    */
```
for eg. if you have add book route you can do:
```
app.post("/addbook", (req, res) => {
  /*    #swagger.parameters['body'] = {
            in: 'body',
            description: 'Book data.',
            required: true,
            schema: {
$ref: "#/components/schemas/someSchema"
            }
        }
    */

    ... rest of the code
```