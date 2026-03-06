<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Keep-ish Notes</title>
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link
      href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap"
      rel="stylesheet"
    />
    <link rel="stylesheet" href="styles.css" />
  </head>
  <body>
    <header class="topbar">
      <div class="brand">
        <span class="logo">📝</span>
        <h1>Keep-ish Notes</h1>
      </div>
      <div class="auth-controls">
        <span id="syncStatus" class="status">Local only</span>
        <button id="signInBtn" class="text-btn">Sign in with Google</button>
        <button id="signOutBtn" class="text-btn hidden">Sign out</button>
      </div>
    </header>

    <main class="page">
      <section class="composer card">
        <input
          id="noteTitle"
          type="text"
          placeholder="Title"
          maxlength="120"
          aria-label="Note title"
        />
        <textarea
          id="noteText"
          placeholder="Take a note... (no text limit)"
          aria-label="Note body"
        ></textarea>
        <div class="composer-actions">
          <button id="addNoteBtn" class="primary">Add note</button>
          <small>Unlimited note text supported.</small>
        </div>
      </section>

      <section id="notesGrid" class="notes-grid" aria-live="polite"></section>
    </main>

    <template id="noteTemplate">
      <article class="note card">
        <h2 class="note-title"></h2>
        <p class="note-body"></p>
        <div class="note-actions">
          <button class="copy-btn">Copy</button>
          <button class="delete-btn">Delete</button>
        </div>
      </article>
    </template>

    <script type="module" src="app.js"></script>
  </body>
</html>
