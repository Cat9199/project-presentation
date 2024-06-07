### Slide 1: Database Setup - Part 1

**Title: Database Setup**

**Content:**
1. **Install MySQL:**
   - Download and install MySQL from the official website or use a package manager.
   - Start the MySQL server.

2. **Create a Database:**
   ```sql
   CREATE DATABASE user_management;
   ```

3. **Create a User and Grant Privileges:**
   ```sql
   CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
   GRANT ALL PRIVILEGES ON user_management.* TO 'username'@'localhost';
   FLUSH PRIVILEGES;
   ```

**Visuals:**
- Screenshots of MySQL installation.
- SQL commands in a code editor or terminal.

### Slide 2: Database Setup - Part 2

**Title: Configure Laravel for the Database**

**Content:**
1. **Install Laravel:**
   ```bash
   composer create-project --prefer-dist laravel/laravel user-management-api
   ```

2. **Configure the .env File:**
   - Open the `.env` file in the root of your Laravel project.
   - Set up the database configuration:
     ```env
     DB_CONNECTION=mysql
     DB_HOST=127.0.0.1
     DB_PORT=3306
     DB_DATABASE=user_management
     DB_USERNAME=username
     DB_PASSWORD=password
     ```

3. **Run Migrations:**
   ```bash
   php artisan migrate
   ```

**Visuals:**
- `.env` file with highlighted database settings.
- Terminal showing migration command and success message.

### Slide 3: Backend Development - Part 1

**Title: User Registration Endpoint**

**Content:**
1. **Create User Model:**
   ```bash
   php artisan make:model User -m
   ```

2. **Define User Schema:**
   - Open the migration file in `database/migrations`.
   - Define the schema:
     ```php
     Schema::create('users', function (Blueprint $table) {
         $table->id();
         $table->string('name');
         $table->string('email')->unique();
         $table->string('phone');
         $table->string('password');
         $table->timestamps();
     });
     ```

3. **Create Auth Controller:**
   ```bash
   php artisan make:controller AuthController
   ```

4. **Add Register Method:**
   ```php
   public function register(Request $request) {
       $validatedData = $request->validate([
           'name' => 'required|string|max:255',
           'email' => 'required|string|email|max:255|unique:users',
           'phone' => 'required|string|max:20',
           'password' => 'required|string|min:8',
       ]);

       $validatedData['password'] = bcrypt($validatedData['password']);
       $user = User::create($validatedData);

       return response()->json(['message' => 'User registered successfully', 'user' => $user], 201);
   }
   ```

**Visuals:**
- Code snippets for migration and controller.
- Terminal showing User model creation.

### Slide 4: Backend Development - Part 2

**Title: Login and Fetch User Details Endpoints**

**Content:**
1. **Login Method in AuthController:**
   ```php
   public function login(Request $request) {
       $credentials = $request->only('email', 'password');

       if (Auth::attempt($credentials)) {
           $user = Auth::user();
           $token = $user->createToken('authToken')->accessToken;
           return response()->json(['message' => 'Login successful', 'token' => $token], 200);
       } else {
           return response()->json(['message' => 'Invalid email or password'], 401);
       }
   }
   ```

2. **Fetch User Details Method:**
   ```php
   public function userDetails() {
       $user = Auth::user();
       return response()->json(['user' => $user], 200);
   }
   ```

3. **Configure Routes:**
   - Add routes in `routes/api.php`:
     ```php
     Route::post('register', [AuthController::class, 'register']);
     Route::post('login', [AuthController::class, 'login']);
     Route::middleware('auth:api')->get('user', [AuthController::class, 'userDetails']);
     ```

**Visuals:**
- Code snippets for controller methods and routes.
- Screenshot of successful API calls in Postman.

### Slide 5: Deployment - Part 1

**Title: Preparing for Deployment**

**Content:**
1. **Set Up Server:**
   - Choose a hosting provider (e.g., AWS, DigitalOcean).
   - Set up a virtual server with a LEMP stack (Linux, Nginx, MySQL, PHP).

2. **Clone the Repository:**
   ```bash
   git clone <repository_url>
   cd user-management-api
   ```

3. **Configure Environment:**
   - Copy `.env.example` to `.env`.
   - Set production database credentials in the `.env` file.

**Visuals:**
- Screenshot from cloud server dashboard.
- Terminal commands for cloning the repository and setting environment variables.

### Slide 6: Deployment - Part 2

**Title: Deploying the Application with Nginx**

**Content:**
1. **Install Dependencies:**
   ```bash
   composer install
   ```

2. **Run Migrations:**
   ```bash
   php artisan migrate --force
   ```

3. **Set Up Nginx Configuration:**
   - Create a new Nginx configuration file at `/etc/nginx/sites-available/user-management-api`:
     ```nginx
     server {
         listen 1234;
         server_name example.com;

         root /var/www/html/user-management-api/public;
         index index.php index.html index.htm;

         location / {
             try_files $uri $uri/ /index.php?$query_string;
         }

         location ~ \.php$ {
             include snippets/fastcgi-php.conf;
             fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
             fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
             include fastcgi_params;
         }

         location ~ /\.ht {
             deny all;
         }
     }
     ```

4. **Enable the Site and Restart Nginx:**
   ```bash
   ln -s /etc/nginx/sites-available/user-management-api /etc/nginx/sites-enabled/
   nginx -t
   systemctl restart nginx
   ```
