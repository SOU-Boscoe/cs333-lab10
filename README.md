# cs333-lab10
The full todo app on your server, and a final project checkpoint

In week 8 you ran the todo app on your laptop: the React front end in one terminal, the Express and MongoDB back end in another. Now both go on your server, so anyone can use it. Then you'll crash the back end on purpose, fix it with some error handling, and write down where your final project is.

If your final project is built on the todo app, this lab puts the first version of your project online. If your project is your own app, do the steps with your own front end and back end folders instead (there's a note at each step). Either way, by the end you have a React front end and an Express + MongoDB back end working together on your server.

Do each step below and answer the questions right here in this README as you go (type your answer under each question).

You need:
- the todo app from week 8 (`333FullToDo-2026.zip` from Slack), running on your laptop
- your server, the same setup as Lab 6: you can `ssh yourfirstname@your-server-address`, pm2 is installed, and your nginx has the `location /api/` block
- MongoDB running on your server (week 6)

Laptop or server? Every step says (laptop) or (server). Your prompt tells you where you are: your laptop's name, or `yourfirstname@` your server. Most mistakes in this lab come from typing a command in the wrong terminal.

You'll want three terminals open the whole lab: two on your laptop (one in `cs333-api-main`, one in `cs333-main`) and one logged in to your server.

Stuck? Look at the Stuck table at the bottom first, then post in the Slack lab thread: the command you ran, the exact error (copy and paste it), and what you already tried.

---

# Part 1: Your repo (laptop)

1. Click "Use this template" to make your own copy of this repo and clone it to your laptop.

2. Copy the two folders from the todo app, `cs333-api-main` and `cs333-main`, into your repo folder. Copy the folders, not the whole `333FullToDo` folder. Don't copy `node_modules` if you can help it; it's fine if you do, `.gitignore` keeps it off GitHub.

Your own project instead? Copy your back end folder and your front end folder in.

3. Make sure it still runs on your laptop, the same as week 8. MongoDB running on your laptop, then:

Laptop terminal 1, the back end:

```bash
cd cs333-api-main
npm install
cp .env.example .env
npm run dev
```

(Windows Command Prompt: `copy .env.example .env`)

Laptop terminal 2, the front end:

```bash
cd cs333-main
npm install
npm run dev
```

Open http://localhost:5173 and add a todo.

Question 1: which terminal is running the back end, and which port is it on? Which one is the front end, and which port is it on? How does the front end's `fetch("/api/")` get to the back end on your laptop? (Hint: `cs333-main/vite.config.js`.)

Commit: `Part 1: todo app in my repo`

---

# Part 2: The back end on your server

Your jokes API from Lab 6 is on port 3000, and the todo back end needs port 3000 too. Only one program can use a port at a time, so the jokes API steps aside. (Your Lab 6 is already handed in, so that's fine.)

4. (server) Log in to your server in your third terminal and stop the jokes API:

```bash
ssh yourfirstname@your-server-address
pm2 stop jokes
pm2 list
```

`jokes` should say `stopped`. Make a folder for the todo back end:

```bash
mkdir ~/todo-api
```

Your back end goes in your home folder, not in `/var/www/html`, same as the jokes API in Lab 6.

5. (laptop) In laptop terminal 1, press Ctrl+C to stop the back end. You should still be in `cs333-api-main`. Copy the back end up, without `node_modules` and without your `.env`:

```bash
scp -r index.js package.json package-lock.json .env.example config models routes yourfirstname@your-server-address:~/todo-api/
```

Using WinSCP instead? Go to `/home/yourfirstname/todo-api` on the right, and drag over those files and folders: `index.js`, `package.json`, `package-lock.json`, `.env.example`, `config`, `models` and `routes`.

Your own project? Copy your back end's files the same way, everything except `node_modules` and `.env`.

6. (server) Install, and make the server's own `.env`:

```bash
cd ~/todo-api
ls
npm install
cp .env.example .env
nano .env
```

In nano, change `NODE_ENV=development` to `NODE_ENV=production`. Leave the other two lines. Save with Ctrl+O, Enter, then Ctrl+X.

Question 2: you made a `.env` on your laptop in step 3, but you didn't copy it up. You made a new one on the server. Why is that a good idea? (Think about what a `.env` holds on a real project, and why it's in `.gitignore`.)

7. (server) Check MongoDB is running on your server:

```bash
systemctl status mongod --no-pager | head -3
```

It should say `active (running)`. (If it doesn't, look at the Stuck table.)

8. (server) Start the back end with pm2. In Lab 6 you started `index.js` by itself. This back end starts with `npm start` (it reads your `.env`, look in `package.json`), so you tell pm2 to run `npm start`:

```bash
pm2 start npm --name todo -- start
pm2 list
pm2 logs todo --lines 20
```

Press Ctrl+C to stop watching the log (that doesn't stop the back end).

Question 3: what do the last lines of the log say? Which environment is it running in, and why?

9. (server) Test it from the server:

```bash
curl http://localhost:3000/
```

You should get `[]`: an empty list, because your server's database has no todos yet.

Then save pm2's list, so the todo back end (and not jokes) comes back if the server restarts:

```bash
pm2 save
```

10. (laptop) Now from the internet. Open `http://your-server-address/api/` in your laptop's browser. You should see `[]` again.

Question 4: the browser asked for `/api/`, and Express has a route for `/`. What took the `/api` off on the way? (You set it up in Lab 6.)

Commit: `Part 2: back end on my server`

---

# Part 3: The front end on your server

11. (laptop) In laptop terminal 2, press Ctrl+C to stop the front end. You should still be in `cs333-main`. Build it, the same two ways as Labs 7, 8 and 9. Way B keeps your other labs where they are, so it's the one I'd pick.

## Way A: straight into /var/www/html

Heads up: this replaces whatever is in `/var/www/html` now.

```bash
npm run build
scp -r dist/* yourfirstname@your-server-address:/var/www/html/
```

Your site: `http://your-server-address`

## Way B: in a folder called todo

```bash
npm run build -- --base=/todo/
```

(server) Make the folder:

```bash
mkdir /var/www/html/todo
```

(laptop) Copy what's inside `dist` into it:

```bash
scp -r dist/* yourfirstname@your-server-address:/var/www/html/todo/
```

Using WinSCP instead? Make a new folder called `todo` inside `/var/www/html` on the right, open it, and drag everything inside your `dist` folder (not the `dist` folder itself) over.

Your site: `http://your-server-address/todo/`

12. (laptop) Open your site. Add two todos, check one off, edit one, and refresh the page. They should all still be there.

Question 5: the front end is in `/var/www/html` and the back end is in `~/todo-api`. Who hands out each one when someone opens your site: nginx, or pm2 and Express? Draw it with arrows in plain text if that helps, like the picture in Lab 6.

13. (server) Look at your todos in the server's MongoDB:

```bash
mongosh cs333 --eval "db.todos.find()"
```

Question 6: are these the todos you made on your laptop in step 3, or the ones you just made on your site? Why?

Commit: `Part 3: front end on my server`

---

# Part 4: Crash it, then fix it

Right now the back end has no error handling at all. Let's see what that does.

14. (laptop) Open `http://your-server-address/api/notanid` in your browser. A todo's id looks like `6af62c65489a1c01f91e8766`, and `notanid` isn't one.

(server) Then look:

```bash
pm2 list
pm2 logs todo --err --lines 15
```

Question 7: what did the browser show? What happened to the `↺` number for `todo` in `pm2 list`? What's the error in the log (the first line that says what went wrong)?

So one bad link crashed your whole back end, for everybody using your site. pm2 started it again, but anyone using it right then got an error.

15. (laptop) The fix. Open `cs333-api-main/routes/routes.js` in VS Code and find the route that starts with `// GET route to retrieve a specific item`. Replace the whole route with this:

```js
// GET route to retrieve a specific item
router.get('/:id', async (req, res) => {
  const itemId = req.params.id
  try {
    const todo = await Todo.findById(itemId)
    if (todo === null) {
      console.log('No todo with id', itemId)
      return res.status(404).send({ error: 'No todo with that id' })
    }
    res.status(200).send(todo)
  } catch (error) {
    console.log('Bad id', itemId, error.message)
    res.status(400).send({ error: 'That is not a todo id' })
  }
})
```

Three things are new:
- `try { ... } catch (error) { ... }`: if anything inside `try` throws an error (like a bad id), the code jumps to `catch` instead of crashing.
- `if (todo === null)`: the id looks right, but there's no todo with it. That's a 404, the same as a page that doesn't exist.
- `return` stops the route there, so it doesn't send a second answer.

16. (laptop) Try it on your laptop first. Start the back end again in laptop terminal 1 (`npm run dev`) and open these in your browser:

- `http://localhost:3000/notanid`
- `http://localhost:3000/6af62c65489a1c01f91e0000` (looks like an id, but isn't one of yours)

Question 8: what does each one show? What does laptop terminal 1 print for each? Is the back end still running?

17. TODO: the `DELETE` route at the bottom of `routes.js` has the same problem. Give it a `try`/`catch` and a 404 the same way. (Hint: `findByIdAndDelete` gives you back `null` when there was nothing to delete.)

18. (laptop) Copy the fixed file up. From `cs333-api-main`:

```bash
scp routes/routes.js yourfirstname@your-server-address:~/todo-api/routes/
```

(server) Restart the back end so it uses the new file:

```bash
pm2 restart todo
```

Run `pm2 list` and look at the `↺` number for `todo` (`pm2 restart` just added one to it). Open `http://your-server-address/api/notanid` again. You should get `{"error":"That is not a todo id"}`. Run `pm2 list` again: the `↺` number should be the same as before, because nothing crashed.

19. Comments and console.logs. Add a comment above each route in `routes.js` saying what it does, in your own words, if it doesn't have one. Add at least one `console.log` of your own somewhere in the back end or front end that helped you see what your code was doing. You can leave your console.logs in, or comment them out with `//`. Either way is fine. (On the server, your back end's console.logs show up in `pm2 logs todo`.)

Commit: `Part 4: error handling`

---

# Part 5: Final project checkpoint

Answer these about your final project. Short answers are fine. This is where you are now, not where you'll end up.

Question 9: is your final project built on the todo app, or is it your own app? What's its GitHub repo link? (If it's built on the todo app, this repo can be it.)

Question 10: what works on your server right now? Give the link.

Question 11: from your planning doc, what are the next three things you're going to build? Which one do you think is the hardest, and what's your plan if it doesn't work?

20. Optional styling. The todo app's look is all in `cs333-main/src/index.css`. Paste this at the bottom, then change the colors and sizes to make it yours (keep the class names, the comments in the code say `className important` for a reason):

```css
body {
  background-color: #f4f1ea;
  font-family: Arial, sans-serif;
}

.todos header h1 {
  color: darkslateblue;
}

.todo {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
}
```

Build and copy it up again (step 11) to see it on your server.

Commit: `Part 5: checkpoint`

---

## Changed your code?

Back end: copy the changed file up from your laptop (`scp`, or WinSCP), then on the server run `pm2 restart todo`. If you added a package, run `npm install` in `~/todo-api` before the restart.

Front end: build again on your laptop and copy `dist/*` up again (step 11).

---

# Hand it in

21. Push this repo to your own GitHub repo: both folders and this README with your answers. `node_modules`, `dist` and `.env` are left out on purpose, `.gitignore` takes care of that.

22. Submit in Moodle: your GitHub repo link and your site link (`http://your-server-address` or `http://your-server-address/todo/`). Leave it up so I can try it.

---

## Stuck?

| You see | What's going on | Fix |
| --- | --- | --- |
| `scp: stat local "config": No such file or directory` | You're not in `cs333-api-main` on your laptop | `cd cs333-api-main`, `ls` should show `config`, `models`, `routes` |
| `scp: ... No such file or directory` about `~/todo-api` | The folder isn't there on the server | Step 4: `mkdir ~/todo-api` |
| `.env: not found` in `pm2 logs todo` | You skipped `cp .env.example .env` on the server | Step 6, then `pm2 restart todo` |
| `EADDRINUSE: address already in use :::3000` | The jokes API is still on port 3000 | `pm2 stop jokes`, `pm2 restart todo`, `pm2 save` |
| `pm2 list` says `errored`, or `↺` keeps going up | The back end crashes when it starts | `pm2 logs todo --err` shows the error |
| `Mongo Connection Error` / `ECONNREFUSED 127.0.0.1:27017` | MongoDB isn't running on your server | `sudo systemctl start mongod`, then `pm2 restart todo` |
| `mongod` says `failed` or keeps stopping | The MongoDB problem from week 6 | Week 6 slides, the steps that fix Mongo, then post in the Slack thread |
| `502 Bad Gateway` at `/api/` | nginx is fine, but the back end isn't running | `pm2 list` and `pm2 logs todo` |
| `404 Not Found` from nginx at `/api/` | The `location /api/` block isn't there, or nginx wasn't reloaded | Lab 6 step 19 |
| `Cannot GET /api/` (from Express) | The last `/` is missing in `proxy_pass` | Lab 6: `proxy_pass http://localhost:3000/;` |
| The site loads but says No todos to display, and adding one does nothing | The front end can't reach the back end | Open the developer tools, Network tab, add a todo, and look at the `/api/` request. 502 means the back end is down |
| Way B: `/todo/` shows a blank page | You built without `--base=/todo/` | `npm run build -- --base=/todo/` and copy `dist/*` into `/var/www/html/todo/` again |
| `scp: ... Permission denied` or `mkdir: ... Permission denied` in `/var/www/html` | `/var/www/html` doesn't belong to you yet | On the server: `sudo chown -R $USER:$USER /var/www/html` (week 6) |
| `/api/notanid` still crashes after your fix | The server still has the old `routes.js`, or you didn't restart | Step 18: copy `routes/routes.js` up again, then `pm2 restart todo` |
