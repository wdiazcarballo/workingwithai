To start developing and documenting your RESTful API project using Swagger UI, you'll follow a few structured steps. This process involves setting up Swagger to work with your existing Node.js and Express application, generating an OpenAPI (Swagger) specification, and integrating Swagger UI to visualize and interact with your API. Here's a step-by-step guide:

### Step 1: Install Swagger-Related Packages

First, you need to install the necessary packages that will help you integrate Swagger into your Node.js project. You can use `swagger-ui-express` and `swagger-jsdoc` for this purpose.

```bash
npm install swagger-ui-express swagger-jsdoc --save
```

### Step 2: Set Up Swagger JSDoc Comments

Before you generate the documentation, you need to annotate your existing API routes with JSDoc comments that `swagger-jsdoc` can understand to generate the OpenAPI specification. Here’s an example of how you might annotate a route in your `userRoutes.js` or `crmRoutes.js` file:

```javascript
/**
 * @swagger
 * /contact:
 *   get:
 *     summary: Retrieve a list of contacts
 *     description: Retrieve a list of all contacts in the CRM system.
 *     responses:
 *       200:
 *         description: A list of contacts.
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/Contact'
 */
```

### Step 3: Create the Swagger Definition

Create a setup file or section in your main application file (`index.js`) where you configure the swagger specification:

```javascript
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'CRM API',
      version: '1.0.0',
      description: 'A simple CRM API',
    },
    servers: [
      {
        url: 'http://localhost:3000',
        description: 'Development server',
      },
    ],
  },
  apis: ['./src/routes/*.js'], // paths to files containing Swagger annotations
};

const swaggerSpec = swaggerJsdoc(options);

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

### Step 4: Serve the API Documentation

Now, your API documentation is accessible at `http://localhost:3000/api-docs`. This URL serves the Swagger UI, which loads your API documentation generated from your route annotations. You can interact with your API directly from this interface.

### Step 5: Test and Iterate

Use Swagger UI to test your API endpoints. This interface allows you to make API calls without using a separate tool like Postman. It’s beneficial for quick testing and verification. Adjust your annotations and logic as necessary based on the results and feedback you receive from testing.

### Conclusion

By following these steps, you integrate Swagger UI into your existing RESTful API project, allowing you to visualize, interact, and develop your API in a structured and efficient manner. This setup not only aids in development but also ensures that your API is well-documented and easier for other developers to understand and use.
