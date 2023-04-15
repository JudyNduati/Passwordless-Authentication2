## Passwordless authentication in practice with Node.js
To demonstrate passwordless authentication in practice with Node.js to users. You will need a basic understanding of HTML and JavaScript to create a front end. Also, you will need [Node.js](https://nodejs.org/en/download) installed on your device.

- You will create a form that collects the user's email address on the login page.
- When the users enter and send their email addresses. A magic link is sent. The Node.js server should generate a unique token and send it to the user's email address along with the link to the login page.
- When the user clicks the link, the server verifies if the token is valid.
- If the token is valid, the server sets a session for the user. Finally, the user gets redirected to the application.

Here is the implementation of passwordless authentication using `HTML`, `JavaScript`, and `Node.js`.

Create an HTML form `index.html` that uses an `index.js`file.

```HTML
<!DOCTYPE html>
<html>
    <head>
        <title>Passwordless Authentication</title>
        <script src="index.js"></script>
    </head>
    <body>
        <h1>Enter your email and get a magic link.</h1>
        <form>
            <div>
                <label for="email_address">Enter your email address</label>
                <input type="email" id="email_address" />
            </div>
            <button type="send" id="send_email">Get magic link</button>
        </form>
    </body>
</html>
```

`index.js`:
```JavaScript
window.onload = () => {
    const submitButton = document.getElementById("send_email");
    const emailInput = document.getElementById("email_address")
    submitButton.addEventListener("click", handleAuth);
    
    /** This function sends a request to the server and sends a magic link to the user.*/

    async function handleAuth() {
      const message = await axios.post("http://localhost:3000/send_magic_link", {
        email: emailInput.value
      });
      return message;
    }
  };

```
### Node.js server setup
In this section, you'll set up the Node.js server. You'll begin by creating an express application and installing a few packages. The packages include `expess`, `body-parser`, `cors`, `nodemailer`, `jsonwebtoken`.

```JavaScript
const PORT = process.env.PORT || 3000;

const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const nodemailer = require('nodemailer');
const jwt = require ('jsonwebtoken');

const app = express();

app.use (cors());
app.use (bodyParser.json());

app.post('/send_magic_link', function (req, res) {

  const email = req.body.email;

    if (!email) {
      res.statusCode(403);
      res.send({
        message: "Invalid email address.",
      });
    }
  
    if (email) {
      res.statusCode(200);
      res.send(email);
    }
    
});

app.get('/authenticate_user', function (req, res) {
   
});

app.get('/welcome', (req, res) {
  res.status(200).send(
    "You are logged in!"
  );

});

const server = app.listen(PORT, () => {
    console.log(`Server running on port ${server.address().port}`);
});
```

### Create a function for sending the magic link
The installed packages allow us to handle incoming requests, parse data, and send emails. Because you installed the `nodemailer`, you will create a transporter to send magic links.

```JavaScript
    const transport = nodeMailer.createTransport({
      host: process.env.EMAIL_HOST,
      port: 587,
      auth: {
          user: process.env.EMAIL_USER,
          pass: process.env.EMAIL_PASSWORD
      }
    });

    // Create email template for magic link
      const emailTemplate = ({ username, link }) => `
        <h2>Hey ${username}</h2>
        <p>Here's the magic link you requested:</p>
        <p>${link}</p>
      `
```

### Generating tokens

When the server receives the email address, It should generate a unique token and store it in a database. The token holds the user's information, then authenticates the user.

```JavaScript
  //Generating a token
      const makeToken = (email) {
        const expirationDate = new Date();
        expirationDate.setHours(new Date().getHours() + 1);
        return jwt.sign({ email, expirationDate }, process.env.JWT_SECRET_KEY);
      };
```

Users should be authenticated and logged in to the app immediately when they click the link.
```JavaScript
app.post("/send_magic_link", function (req, res) {
  const { email } = req.body;
  if (!email) {
    res.status(403);
    res.send({
      message: "Invalid email address.",
    });
  }
  const token = makeToken(email);
  const mailOptions = {
    from: "You Know",
    html: emailTemplate({
      email,
      link: `http://localhost:3000/authenticate_user?token=${token}`,
    }),
    subject: "Your Magic Link",
    to: email,
  };
  return transport.sendMail(mailOptions, (error) {
    if (error) {
      res.status(403);
      res.send("Can't send email.");
    } else {
      res.status(200);
      res.send(`Magic link sent. : http://localhost:3000/authenticate_user?token=${token}`);
    }
  });
});
```

Next, we will need an authentication method.
```JavaScript
app.get('/authenticate_user', (req, res) {
  isAuthenticated(req, res)
});
```

The authentication method is called once the token is taken from the front end.
```JavaScript
const isAuthenticated = (req, res) => {  const { token } = req.query
  if (!token) {
    res.status(403)
    res.send("Can't verify user.")
    return
  }
  let decoded
  try {
    decoded = jwt.verify(token, process.env.JWT_SECRET_KEY)
  }
  catch {
    res.status(403)
    res.send("Invalid auth credentials.")
    return
  }
  if (!decoded.hasOwnProperty("email") || !decoded.hasOwnProperty("expirationDate")) {
    res.status(403)
    res.send("Invalid auth credentials.")
    return
  }
  const { expirationDate } = decoded
  if (expirationDate < new Date()) {
    res.status(403)
    res.send("Token has expired.")
    return
  }
  res.status(200)
  res.send("User has been validated.")

  const user = users.find(user => user.token === token);

    if (user) {
    req.session.email = user.email;
    // Get the token from memory
    users.splice(users.indexOf(user), 1);
    res.redirect('http://localhost:3000/welcome');

  } else {
    res.send('Not authorized.');
  }

}
```
When the user clicks the link sent via email, the server should be able to verify the token. This involves comparing it with the secret code in the database. If the tokens match, the user gets authenticated by decoding it with the secret key that created it. If tokens do not match, the system returns an error.