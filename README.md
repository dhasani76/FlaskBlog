
                         #FLASK BLOG APP DESIGN

1. OVERVIEW
-----------
This is a full-featured, secure Flask-based blogging platform ("Blog Capstone").
The application includes user authentication, dynamic blog post creation,
rich text editing, comment threads with Gravatar integration, and a contact 
form that sends emails using SMTP.

2. SYSTEM ARCHITECTURE & TECH STACK
-----------------------------------
- Back-end Framework: Flask (Python)
- Database ORM: SQLAlchemy with Flask-SQLAlchemy
- Database Engine: Configurable (SQLite for local, PostgreSQL or others via DB_URI)
- Frontend Styling: Bootstrap 5 (integrated using Flask-Bootstrap5)
- Text Editor: CKEditor (Flask-CKEditor) for rich-text posts and comments
- User Authentication: Flask-Login (handles user sessions, logins, and logouts)
- Passwords Hashing: Werkzeug Security (PBKDF2/SHA256 hashing)
- User Avatars: Gravatar (Flask-Gravatar)
- Forms & Validation: WTForms / Flask-WTF with email validator support
- Environment Variables: python-dotenv (reads from .env)
- Contact Emailing: Python's smtplib (SMTP SSL)

3. DATABASE SCHEMA & RELATIONSHIPS
----------------------------------
The database uses SQLAlchemy models mapping three main tables:

  [Users Table]
  +----+----------+-------------------+----------+
  | id |   name   |       email       | password |
  +----+----------+-------------------+----------+
         |                                 |
         | (1-to-many posts)               | (1-to-many comments)
         v                                 v
  [BlogPosts Table] --------------> [Comments Table]
  +----+-----------+-----------+    +----+------+-----------+---------+
  | id | author_id |   title   |    | id | text | author_id | post_id |
  |    | subtitle  |   date    |    +----+------+-----------+---------+
  |    |   body    |  img_url  |             ^
  +----+-----------+-----------+             | (1-to-many post comments)
                      |                      |
                      +----------------------+

- User (users):
  - id (Integer, primary_key)
  - email (String(100), unique)
  - password (String(250) - hashed)
  - name (String(100))
  - Relationship 'posts': points to BlogPost model (back_populates="author")
  - Relationship 'comments': points to Comment model (back_populates="comment_author")

- BlogPost (blog_posts):
  - id (Integer, primary_key)
  - author_id (Integer, ForeignKey to users.id)
  - title (String(250), unique, nullable=False)
  - subtitle (String(250), nullable=False)
  - date (String(250), nullable=False)
  - body (Text, nullable=False)
  - img_url (String(250), nullable=False)
  - Relationship 'author': points to User model (back_populates="posts")
  - Relationship 'comments': points to Comment model (back_populates="parent_post")

- Comment (comments):
  - id (Integer, primary_key)
  - text (Text, nullable=False)
  - author_id (Integer, ForeignKey to users.id)
  - post_id (Integer, ForeignKey to blog_posts.id)
  - Relationship 'comment_author': points to User model (back_populates="comments")
  - Relationship 'parent_post': points to BlogPost model (back_populates="comments")

4. AUTHENTICATION & ACCESS CONTROL
----------------------------------
- Password Security:
  - Registered passwords are hashed before storing in the database using 
    werkzeug.security.generate_password_hash.
  - Matches are checked using werkzeug.security.check_password_hash on login.
- Session Management:
  - Flask-Login tracks user sessions, using a `load_user` callback.
- Authorization Roles:
  - Regular Users: Can register, login, view posts, write comments.
  - Admin (User ID = 1): Unique status to create, edit, or delete blog posts.
  - Custom decorator @admin_only is implemented to wrap admin routes and abort 
    with a HTTP 403 Forbidden error for unauthorized users.

5. API ROUTING & VIEWS
----------------------
- GET /
    Renders 'index.html', showing a feed of all blog posts in reverse chronological/standard order.
- GET, POST /register
    Renders 'register.html'. Standard user registration form. Saves new users to DB with hashed passwords.
- GET, POST /login
    Renders 'login.html'. Standard user login form. Stores active user session.
- GET /logout
    Ends active user session. Redirects to homepage.
- GET, POST /post/<post_id>
    Renders 'post.html'. Shows selected blog post, list of related comments, and comment editor.
- GET, POST /new-post  [@admin_only]
    Renders 'make-post.html'. CKEditor form to create a new blog post.
- GET, POST /edit-post/<post_id>  [@admin_only]
    Renders 'make-post.html'. Modifies existing post data.
- GET /delete/<post_id>  [@admin_only]
    Deletes the post from the database.
- GET /about
    Renders static 'about.html' page.
- GET, POST /contact
    Renders 'contact.html'. Handles contact form inputs (name, email, phone, message).
    If method is POST, sends an email notification via Google SMTP SSL.

6. FORM VALIDATION
------------------
Handled using WTForms in forms.py:
- CreatePostForm: Fields for Title, Subtitle, Image URL (URL validated), and CKEditor rich-text body.
- RegisterForm: Fields for Name, Email (validated), and Password (min 8 chars).
- LoginForm: Fields for Email (validated) and Password (min 8 chars).
- CommentForm: CKEditor-based text input area.

7. ENVIRONMENT CONFIGURATION
----------------------------
The application reads from a local `.env` file containing:
- SECRET_KEY: Secret key for session cookies.
- DB_URI: Database URL (e.g. SQLite path or PostgreSQL connection string).
- SENDER_EMAIL: Sender email address for SMTP mailing.
- PASSWORD: Gmail app password or login password for SENDER_EMAIL.
- RECEIVER_EMAIL: Recipient email address to receive contact form inquiries.

8. ADDITIONAL FEATURES
----------------------
- Gravatar Integration: Generates profile pictures/avatars for commentators 
  based on their registered email addresses.
- Responsive Design: Uses Clean Blog theme styles wrapped in Bootstrap 5 layouts.
================================================================================
