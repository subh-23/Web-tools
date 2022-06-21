
### Step 1 (Install additional dependencies)

**NPM packages:** passport, passport-local, bcrypt, express-session, express-flash, method-override

```bash
npm i passport passport-local bcrypt express-session express-flash method-override
```

### Step 2  (Modify/add in the main server file / index.js)

- require packages
```bash
const bcrypt = require('bcrypt')
const passport = require('passport')
const flash = require('express-flash')
const session = require('express-session')
const methodOverride = require('method-override')
const initializePassport = require('./passport-config')
```

- initializing passport
```bash
initializePassport(passport)
```
- set-up middle-wares
```bash
app.use(flash())
app.use(session({
    secret: "Secret",
    resave: false,
    saveUninitialized: false
}))
app.use(passport.initialize())
app.use(passport.session())
app.use(methodOverride('_method'))
```

- main login functionallity
```bash
//for rendering register form
app.get('/register', checkNotAuthenticated, (req, res) => {
    res.render('register.ejs')
})
//for rendering login form 
app.get('/login', checkNotAuthenticated, (req, res) => {
    res.render('login.ejs')
})
//for registration 
app.post('/register', checkNotAuthenticated, async (req, res) => {
    try {
        const hashedPassword = await bcrypt.hash(req.body.password, 10)
        //adding to database
        const user = await Users.findOne({ username: req.body.username })
        if (user) return res.status(400).send("User Already Exists")
        await Users.create({
            name: req.body.name,
            username: req.body.username,
            password: hashedPassword
        })
        res.redirect('/login')
    } catch {
        res.redirect('/register')
    }
    // console.log(users)
})
//for authenticate and login
app.post('/login', checkNotAuthenticated, passport.authenticate('local', {
    successRedirect: '/',
    failureRedirect: '/login',
    failureFlash: true
}))
//for logout
app.delete('/logout', function (req, res, next) {
    req.logout(function (err) {
        if (err) { return next(err); }
        res.redirect('/');
    });
});
```

- accessing main index pages (authentication barrier)
```bash
app.get('/', checkAuthenticated, (req, res) => {
    res.render('index');
})
app.post('/', checkAuthenticated, (req, res) => {
    //additional work
})
```


- mandatory boiler-plate
```bash
function checkAuthenticated(req, res, next) {
    if (req.isAuthenticated()) {
        return next()
    }
    res.redirect('/login')
}
function checkNotAuthenticated(req, res, next) {
    if (req.isAuthenticated()) {
        return res.redirect('/')
    }
    next()
}
```
### Step 3 (create a passport config file and copy-paste it)
- import files 
```bash
const LocalStrategy = require('passport-local').Strategy
const User = require('./model/users')
const bcrypt = require('bcrypt')
```
- createing function for authenticate (make sure you're taking input from login and registration form as username and password)
```bash
function initialize(passport) {
    passport.use(
        new LocalStrategy(async (username, password, done) => {
            try {
                const user = await User.findOne({ username })
                if (!user) return done(null, false);
                if (await bcrypt.compare(password, user.password)) return done(null, user)
                return done(null, false, { message: 'Password incorrect' })
            } catch (error) {
                return done(error, false)
            }
        }))
```
- mandatory boilerplate at the end
```bash
    passport.serializeUser((user, done) => {
        done(null, user.id)
    })
    passport.deserializeUser(async (id, done) => {
        try {
            const user = await User.findById(id)
            done(null, user)
        } catch (error) {
            done(error, false)
        }
    })
}

module.exports = initialize
```

## Tadaa!!
