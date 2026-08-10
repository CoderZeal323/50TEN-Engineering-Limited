# 50TEN Engineering Ltd, Company Website

A site for 50TEN Engineering Ltd covering About, Services, Industries, Why Choose Us, HSE, Team, the Academy, a Blog, Trainee Reviews, a Newsletter signup, and Contact, plus a login-gated admin panel for managing blog posts.

Two files, no build step, no framework, no dependencies to install locally:

- `index.html`, the public site
- `admin.html`, the blog admin panel (requires login)

## Before you deploy: add your Supabase key

The blog, newsletter, trainee reviews, and admin panel all read from and write to Supabase. Before this site is live, you need to add your Supabase anon (public) key **in both files**.

1. Open `index.html`, search for `PASTE_YOUR_SUPABASE_ANON_PUBLIC_KEY_HERE`, replace it
2. Open `admin.html`, do the same, it has its own copy of the same line
3. Get the key from Supabase: Dashboard -> Settings -> API -> "Project API keys" -> the one labeled `anon` `public`

**Do not use the `service_role` key here.** That key bypasses all database security rules. If it ends up in either file, anyone visiting the site could read or delete everything in your database. Only the `anon` key belongs in client-side code like this.

## Supabase setup

Run this once in Supabase: Dashboard -> SQL Editor -> New query -> paste and run.

```sql
create table if not exists posts (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  slug text not null unique,
  excerpt text,
  content text not null,
  cover_image text,
  published boolean not null default true,
  created_at timestamptz not null default now()
);

alter table posts enable row level security;

-- Anyone can read published posts (the public blog page)
create policy "Public can read published posts"
on posts for select
using (published = true);

-- Only logged-in admin users can read every post, including drafts
create policy "Authenticated can read all posts"
on posts for select
to authenticated
using (true);

-- Only logged-in admin users can create, edit, or delete posts
create policy "Authenticated can insert posts"
on posts for insert
to authenticated
with check (true);

create policy "Authenticated can update posts"
on posts for update
to authenticated
using (true);

create policy "Authenticated can delete posts"
on posts for delete
to authenticated
using (true);

create table if not exists subscribers (
  id uuid primary key default gen_random_uuid(),
  email text not null unique,
  created_at timestamptz not null default now()
);

alter table subscribers enable row level security;

create policy "Public can subscribe"
on subscribers for insert
with check (true);
```

This also assumes the `review:` key-based storage from the trainee reviews section is already working, since that was set up earlier and doesn't need new tables.

### Creating your admin login

The admin panel uses Supabase's own login system, so there's nothing extra to build for authentication.

1. Supabase Dashboard -> Authentication -> Users -> Add user
2. Set an email and password for yourself (and one each for Gideon or anyone else who should be able to post)
3. That's it, those credentials log into `admin.html`

### Publishing a blog post

Open `admin.html` (once it's deployed, this will be something like `yoursite.pages.dev/admin.html`), log in, and use the "New Post" form. Fill in a title, a URL-safe slug (e.g. `first-cohort-graduates`), an excerpt, and the full content, then hit Save Post. Untick "Published" to save something as a draft without it going live yet. Existing posts can be edited or deleted from the list below the form.

**Note on the admin page's privacy:** `admin.html` isn't hidden or secret, if someone knows or guesses the URL, they'll see the login screen. That's fine, the login itself is what protects the actual data, nobody can create, edit, or delete a post without valid Supabase credentials. This is the same model most small admin panels use.

### Sharing posts to social media

Each post has its own shareable link (e.g. `yoursite.com/?post=first-cohort-graduates`). Opening a post on the site shows LinkedIn and Facebook share buttons that open a prefilled share dialog for that link, one click to post it to either platform.

True automatic posting (no click needed) is a separate, harder project: Facebook is workable with a Meta Developer app and a Page access token, LinkedIn requires their approval to post to an organization page via API, which isn't guaranteed for a small business. Worth revisiting later if the one-click approach feels like too much friction.

### Checking your newsletter list

Subscribers can add themselves through the site, but nobody, including visitors, can read the list back through the site itself. To see who's subscribed, go to Supabase -> Table Editor -> `subscribers`.

## Pushing to GitHub

1. In your GitHub repo, click "Add file" -> "Upload files"
2. Drag in `index.html`, `admin.html`, and this `README.md`
3. Commit directly to `main`

If you'd rather use git from a terminal instead:

```bash
git add index.html admin.html README.md
git commit -m "Add 50TEN Engineering site and admin panel"
git push
```

## Hosting on Cloudflare Pages

1. Cloudflare dashboard -> Workers & Pages -> Create -> Pages -> Connect to Git
2. Authorize GitHub, select your repo
3. Build settings: leave "Build command" blank, set "Build output directory" to wherever the files live (`/` if they're at the repo root, or the folder name if they're in a subfolder)
4. Save and Deploy

Cloudflare gives you a `*.pages.dev` URL right away. A custom domain can be attached afterward from the same project's settings.

## Things still to finish

- Add a real photo for John Bright, Zilpah Idaa, and Gideon in the Team section
- Confirm and add role titles for Zilpah Idaa and Gideon (currently marked "Role to be confirmed")
- Add your Supabase anon key to both `index.html` and `admin.html` (see above)
- Create admin login credentials in Supabase Authentication for yourself and anyone else posting
- Write your first blog post through the admin panel once the site is live
