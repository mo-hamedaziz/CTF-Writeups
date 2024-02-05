# DiceCTF 2024 Quals Writeup
![image](https://github.com/mo-hamedaziz/CTF-Writeups/blob/06973f1edd4a587c902b46304436578890393e33/DiceCTF/2024-Quals/assets/dicectf-logo.png)
## Event Info
Time: Fri, 02 Feb. 2024, 21:00 UTC — Sun, 04 Feb. 2024, 21:00 UTC<br>
Place: Online<br>
Format: Jeopardy<br>
Official URL: https://ctf.dicega.ng/<br>
Rating weight: 54.40 (updated on 05/02/2024)<br>
<br>
## Preface<br>
This has 37 challenges in total: 8 crypto, 9 misc, 6 pwn, 6 rev and 8 web. <br>
In this writeup, we'll discuss 3 web challenges.<br>
## Web:
### <br>web/dicedicegoose<br>
![image](https://github.com/mo-hamedaziz/CTF-Writeups/blob/364aab1b015ff96a7f8e5229ee139e9dd43cf0b0/DiceCTF/2024-Quals/assets/dicedicegoose.png)<br>
This is a simple web game where you can move the dice with WASD keys and you need to catch the black square (the goose). In order to win, the goose and the dice have to overlap, then an alert pops up asking for your name and after submitting you get your score.<br><br>
![image](https://github.com/mo-hamedaziz/CTF-Writeups/assets/114874129/0beb3e39-0bf3-44ba-9113-3d97013ef122)<br><br>
When reading the source code, you can find a function,called **win**, that prints the flag:<br>
```
function win(history) {
    const code = encode(history) + ";" + prompt("Name?");

    const saveURL = location.origin + "?code=" + code;
    displaywrapper.classList.remove("hidden");

    const score = history.length;

    display.children[1].innerHTML = "Your score was: <b>" + score + "</b>";
    display.children[2].href =
      "https://twitter.com/intent/tweet?text=" +
      encodeURIComponent(
        "Can you beat my score of " + score + " in Dice Dice Goose?",
      ) +
      "&url=" +
      encodeURIComponent(saveURL);

    if (score === 9) log("flag: dice{pr0_duck_gam3r_" + encode(history) + "}");
  }
```
Specifically, the following line that contains the flag format (the first half of the flag) and prints it to the console:<br>
```
if (score === 9) log("flag: dice{pr0_duck_gam3r_" + encode(history) + "}");
```
This line means if the goose can be caught in 9 moves, then the flag will be printed in the console.<br>So basically the idea is catching the goose in just 9 moves. However, even in ideal conditions, it's impossible to do that by playing the game normally (because of the green barrier which is 9 blocks tall).<br>
In addition, the goose movement is controlled by a random value generated for the switch statement:<br>
```
do {
      nxt = [goose[0], goose[1]];
      switch (Math.floor(4 * Math.random())) {
        case 0:
          nxt[0]--;
          break;
        case 1:
          nxt[1]--;
          break;
        case 2:
          nxt[0]++;
          break;
        case 3:
          nxt[1]++;
          break;
      }
    } while (!isValid(nxt));

    goose = nxt;
```
So replacing the score with a hard-coded value is all that's needed for the solve.<br>
In the **win** function, the score is set using this line<br>
```
const score = history.length;
```
As it's name suggests, history is an array that stores the dice's moves. The idea is clear now, the **win** function must run when the history length is exaclty equal to 9.<br>In order to do that, we have to move the dice 9 times and the execute the **win** function using the console:<br>
![image](https://github.com/mo-hamedaziz/CTF-Writeups/assets/114874129/b495f0e5-6d21-47ab-9d54-0803e474721a)<br>
In the console, we can see that the history length is now equal to 9.<br>
![image](https://github.com/mo-hamedaziz/CTF-Writeups/assets/114874129/827a5ba9-e74d-4e1f-8a02-4620a372a2db)<br>
Now we can execute the **win** function by typing ``` win(history) ``` in the console.It prints the flag just after submitting our name:<br>
![image](https://github.com/mo-hamedaziz/CTF-Writeups/assets/114874129/0c4b27ae-c509-47b3-8350-fcc65221e917)
<br><br>
### <br>web/funnylogin<br>
![image](https://github.com/mo-hamedaziz/CTF-Writeups/blob/364aab1b015ff96a7f8e5229ee139e9dd43cf0b0/DiceCTF/2024-Quals/assets/funnylogin.png)<br><br>
The application is just a login page. The challenge goal is to get the flag once a valid user is logged as an admin.<br><br>
![image](https://github.com/mo-hamedaziz/CTF-Writeups/assets/114874129/9311e3ec-88a2-4ab6-88a6-a7685b2cdb19)<br><br>
The challenge comes with a compressed file that contains the source code.<br>
![image](https://github.com/mo-hamedaziz/CTF-Writeups/assets/114874129/7d3f60a6-f1ba-4d40-a4c8-72c02cc24795)<br>
This web app have a single route that is /api/login with a POST request. Let's break down the **app.js** file:
Step 1- Creating and SQL table called **users** with **id, username and password** columns:
```
const db = require('better-sqlite3')('db.sqlite3');
db.exec(`DROP TABLE IF EXISTS users;`);
db.exec(`CREATE TABLE users(
    id INTEGER PRIMARY KEY,
    username TEXT,
    password TEXT
);`);
```
Step 2- Generating 10000 random users and inserting them in the table:
```
const users = [...Array(100_000)].map(() => ({ user: `user-${crypto.randomUUID()}`, pass: crypto.randomBytes(8).toString("hex") }));
db.exec(`INSERT INTO users (id, username, password) VALUES ${users.map((u,i) => `(${i}, '${u.user}', '${u.pass}')`).join(", ")}`);
```
The goal of generating such a big number of users with random usernames is to make the bruteforce impossible.<br><br>
Step 3- Set a random user as an admin:
```
const isAdmin = {};
const newAdmin = users[Math.floor(Math.random() * users.length)];
isAdmin[newAdmin.user] = true;
```
It's important to note that the admin if randomly defined at **runtime**. The goal of selecting the admin randomly is to make the username of the admin unguessable.<br><br>
Step 4- Get the username and password from the input:
```
const { user, pass } = req.body;
const query = `SELECT id FROM users WHERE username = '${user}' AND password = '${pass}';`;
```
This query is vulnerable to SQL injection due to the concatenation of user and pass constants from the request body, and the absence of input control or sanitization. So we may think of an SQL injection attack, but the problem is we don’t know what user has isAdmin true because it's random and it's defined at runtime.<br><br>
Step 5- Check the if the **user id** exists and if **isAdmin[user]==true**, if both conditions are true then we are redirected to the flag:
```
try {
        const id = db.prepare(query).get()?.id;
        if (!id) {
            return res.redirect("/?message=Incorrect username or password");
        }

        if (users[id] && isAdmin[user]) {
            return res.redirect("/?flag=" + encodeURIComponent(FLAG));
        }
        return res.redirect("/?message=This system is currently only available to admins...");
    }
    catch {
        return res.redirect("/?message=Nice try...");
    }
```
Thus we can get the flag if:<br>1/The user credential provided should return an id<br>2/The user should have admin permission<br><br>
The trick that comes to mind is using **JavaScript prototype inheritance**. (You can read more about it [**here**](https://portswigger.net/web-security/prototype-pollution))<br>
In fact, JavaScript is based on prototypes: Each Object has an attribute called ```'__proto__'```.<br>
![image](https://github.com/mo-hamedaziz/CTF-Writeups/assets/114874129/a2d71137-abc2-451f-8630-75edd708f8ba)<br>
The idea is clear now, since isAdmin is JavaScript Object ```const isAdmin = {};``` then we'll get ```isAdmin[__prototype__]=true```.<br>
The most obvious solution is to pass ```__proto__``` as **username**. Thus, the SQL is now possible in the password field.<br>
The easiest payload to pass as **password** is ```whatever' or id=123; --```
![image](https://github.com/mo-hamedaziz/CTF-Writeups/assets/114874129/32012ec2-e078-449c-abb2-14bb1fcda831)
<br><br>
We get our flag after submitting:<br>![image](https://github.com/mo-hamedaziz/CTF-Writeups/assets/114874129/d05b9ffa-9957-4752-9f19-8f35e8c48351)
### <br>web/gpwaf<br>
![image](https://github.com/mo-hamedaziz/CTF-Writeups/blob/364aab1b015ff96a7f8e5229ee139e9dd43cf0b0/DiceCTF/2024-Quals/assets/gpwaf.png)

## <br><br> Conclusion
I hope you enjoy and learn something, and if there are any mistakes, please feel free to point out private messages and emails, thank you very much!<br>
My linkedin account: https://www.linkedin.com/in/mohamed-aziz-bchini/<br>
My email: mohamedaziz0801@gmail.com<br>
