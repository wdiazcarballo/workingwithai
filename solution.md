Here are the upgraded source code files:

### 1. `index.js` (Updated to include environment variable usage and error handling middleware)

```javascript
import express from 'express';
import mongoose from 'mongoose';
import routes from './src/routes/userRoutes.js';
import jwt from 'jsonwebtoken';
import dotenv from 'dotenv';

dotenv.config();

const app = express();
const PORT = process.env.PORT || 3000;

// mongoose connection
mongoose.Promise = global.Promise;
mongoose.connect(process.env.DB_URL || 'mongodb://localhost:27017/CRMdb');

// For parsing application/json
app.use(express.json());
// For parsing application/x-www-form-urlencoded
app.use(express.urlencoded({ extended: true }));
app.use(express.static("public"));

// JWT setup
app.use((req, res, next) => {
    if (req.headers && req.headers.authorization && req.headers.authorization.split(' ')[0] === 'JWT') {
        jwt.verify(req.headers.authorization.split(' ')[1], process.env.JWT_SECRET, (err, decode) => {
            if (err) req.user = undefined;
            req.user = decode;
            next();
        });
    } else {
        req.user = undefined;
        next();
    }
});

// Routes
routes(app);

// Error handling middleware
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).send('Something broke!');
});

app.get('/', (req, res) => 
    res.send(`Node and express server is running on port ${PORT}`)
);

app.listen(PORT, '0.0.0.0', () => 
    console.log(`Your server is running on port ${PORT}`)
);
```

### 2. `userRoutes.js` (Updated to include input validation)

```javascript
import { addNewContact, getContacts, getContactWithID, updateContact, deleteContact } from '../controllers/crmController.js';
import { login, register, loginRequired, getUser } from '../controllers/userController.js';
import { check } from 'express-validator';

const routes = (app) => {
    app.route('/contact')
    // get all contacts
    .get((req,res, next) => {
        // middleware
        console.log(`Request from: ${req.originalUrl}`)
        console.log(`Request type: ${req.method}`)
        next();  
    }, loginRequired,getContacts)

    // post a new contact
    .post(loginRequired, [
        check('firstName').not().isEmpty().withMessage('First name is required'),
        check('lastName').not().isEmpty().withMessage('Last name is required'),
        check('email').isEmail().withMessage('Valid email is required')
    ], addNewContact);

    app.route('/contact/:contactId')
    // get specific contact
    .get(loginRequired, getContactWithID)

    .put(loginRequired, [
        check('firstName').not().isEmpty().withMessage('First name is required'),
        check('lastName').not().isEmpty().withMessage('Last name is required'),
        check('email').isEmail().withMessage('Valid email is required')
    ], updateContact)

    .delete(loginRequired, deleteContact)

    app.route('/auth/register')
        .post([
            check('username').not().isEmpty().withMessage('Username is required'),
            check('email').isEmail().withMessage('Valid email is required'),
            check('password').isLength({ min: 6 }).withMessage('Password must be at least 6 characters long')
        ], register);
    
    app.route('/login')
        .post([
            check('email').isEmail().withMessage('Valid email is required'),
            check('password').not().isEmpty().withMessage('Password is required')
        ], login);
}

export default routes;
```

### 3. `userController.js` (Updated to include password hashing and input validation)

```javascript
import mongoose from 'mongoose';
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';
import { UserSchema } from '../models/userModels.js';
import { validationResult } from 'express-validator';

const User = mongoose.model('User', UserSchema);

export const getUser = async (req, res) => {
    try {
        const user = await User.find({});
        res.json(user);
    } catch (err) {
        res.status(400).send(err);
    }
};

export const loginRequired = (req, res, next) => {
    try {
        if (req.user) {
            next();
        } else {
            return res.status(401).json({ message: 'Unauthenticated User' });
        }
    } catch (err) {
        return res.status(500).send({ message: 'Internal server error' });
    }
};

export const register = async (req, res) => {
    try {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({ errors: errors.array() });


        }

        const saltRounds = 10;
        const newUser = new User(req.body);
        const hashedPassword = await bcrypt.hash(req.body.password, saltRounds);
        newUser.hashPassword = hashedPassword;
        const savedUser = await newUser.save();
        return res.json(savedUser);
    } catch (err) {
        return res.status(400).send({
            message: err.message
        });
    }
};

export const login = async (req, res) => {
    try {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({ errors: errors.array() });
        }

        const user = await User.findOne({ email: req.body.email });
        if (!user) {
            return res.status(401).json({ message: 'Authentication failed. No user found' });
        }
        if (!user.comparePassword(req.body.password, user.hashPassword)) {
            return res.status(401).json({ message: 'Authentication failed. Wrong password' });
        }
        const token = jwt.sign({ email: user.email, username: user.username, _id: user.id }, process.env.JWT_SECRET);
        return res.json({ token });
    } catch (err) {
        return res.status(500).send({ message: 'Internal server error' });
    }
};
```

These updated files incorporate improvements in authentication, error handling, input validation, and the use of environment variables. You can replace the corresponding files in your project with these updated versions.
