# 🌟 CGSS Shift Management Project Documentation 🌟

## 📋 Table of Contents
- [Introduction](#introduction)
- [Project Overview](#project-overview)
- [Summary & Explanation](#summary--explanation)
- [Initial Problem](#initial-problem)
- [Host Flask Application](#host-flask-application)
- [Excel/CSV Upload & Editing](#excelcsv-upload--editing)
- [Hide Start and End Times](#hide-start-and-end-times)
- [Editable Board with Dropdowns](#editable-board-with-dropdowns)
- [Dockerize the Project](#dockerize-the-project)
- [GitHub Actions Automation](#github-actions-automation)
- [Jenkins Automation](#jenkins-automation)
- [Terraform Script](#terraform-script)
- [Off Days Management](#off-days-management)
- [Docker on GCP with Terraform](#docker-on-gcp-with-terraform)
- [SSH Authentication](#ssh-authentication)
- [Deployment Summary](#deployment-summary)
- [Kubernetes Cluster Setup](#kubernetes-cluster-setup)
- [Deploy TeamCity on Kubernetes](#deploy-teamcity-on-kubernetes)
- [Deploy Flask App with TeamCity](#deploy-flask-app-with-teamcity)
- [Deploy Flask App with TeamCity & Docker](#deploy-flask-app-with-teamcity--docker)
- [Ansible Automation](#ansible-automation)
- [How to Add Photos](#how-to-add-photos)
- [Conclusion](#conclusion)

## 📖 Introduction
The CGSS Shift Management project transforms a manual Excel-based shift scheduling process into a scalable, cloud-native Flask web application. The system automates employee scheduling, absence management, and notifications, with role-based access control (RBAC) for admins and regular users. Deployed on Google Cloud Platform (GCP) using Docker, Kubernetes, TeamCity, and now Ansible for automation, it ensures reliability and scalability. This documentation provides a detailed, organized, and comprehensive step-by-step account of all tasks, including the creation of a Kubernetes cluster with QEMU/KVM, TeamCity installation, Ansible automation, and related deployment efforts, reflecting the extensive effort invested over several weeks and months.

## 🎯 Project Overview
### 1. Project Overview (Topic & Goal)
The project is a Flask-based web application intended to manage:
![Login Screenshot](docs/login.png)
*Caption: Login page demonstrating user authentication for CGSS Shift Management.*
- User Authentication (Login/Logout) 🔒: Secure access for all users.
- User Management (Create, Update, Delete users) 👥: Admin-only functionality to handle user accounts.
- Employee and Shift Management 📅:
  - Scheduling and assigning work shifts to employees.
  - Viewing and editing employee details.
  - Managing shifts (date, time, assignments).

## 📝 Summary & Explanation
### Core Functionality Clearly Explained
The application allows administrators and regular users to access it securely. Admins can manage users, employees, and shifts. Regular users can view their assigned schedules and related information.

### Simple User Scenario (Use Case Clearly Explained)
#### A. Admin User (amine) 🧑‍💼
- **Login**: Access at `http://localhost:5000/login` with Admin credentials (Username: `amine`, Password: `amine`).
- **After Login**:
  - View dashboard 📊.
  - Create and manage other users (admin, user roles).
  - Assign shifts to employees 🕒.
  - View and manage employee records 📋.
- **Logout**: Click the logout option provided 🚪.

#### B. Regular User 👷
- **Login**: Access at `http://localhost:5000/login` with regular user credentials.
- **After Login**:
  - View their assigned shifts 📅.
  - See personal employee details ℹ️.
  - No administrative management features.
- **Logout**: Click the logout option when finished 🚪.

### Simple Example Workflow
1. Login as Admin (amine/amine) 🔐
2. Add new employees 👥
3. Assign shifts to employees 📅
4. Logout and login as normal user to verify visibility restrictions 👀

### Step-by-Step Guide to Clearly Use the Application
#### Step 1: Run the Application
```bash
cd C:\Users\user\OneDrive\Bureau\adc_webapp-main\adc_webapp-main
.\venv\Scripts\activate
flask run
```
Open browser at: `http://localhost:5000` 🌐

#### Step 2: Login Clearly and Simply
- Username: `amine`
- Password: `amine`

#### Step 2: Use Dashboard and Features (Explicitly Explained)
Navigate via sidebar or dashboard menus:
- **Users** 👥: Create, edit, and manage user accounts.
- **Employees** 🧑‍💼: Manage details, add/remove employees.
- **Shifts** 📅: Schedule and assign shifts to employees.

#### Step 3: Logout Explicitly
After completing work, log out via the logout button 🚪.

### A Quick Summary for Understanding Your Application
| Feature               | Description (clearly)                                | Admin | User |
|-----------------------|-----------------------------------------------------|-------|------|
| **Login/Logout** 🔒   | Securely log in/out using encrypted passwords.      | ✅    | ✅   |
| **Manage Users** 👥   | Create, edit, delete users (Admins Only)            | ✅    |      |
| **Manage Employees** 🧑‍💼 | Add, update, view employee information         | ✅    |      |
| **Shift Management** 📅 | Assign shifts to employees and manage shifts.      | ✅    |      |
| **View Shifts** 👀    | Users and Admins view their assigned shifts.        | ✅    | ✅   |
| **Role-Based** 🔐     | Admin can manage everything; Users limited view     | ✅    |      |

### Why the Login Failed Initially (Problem Explanation)
The Flask application stores passwords using the Fernet encryption method (from the `cryptography.fernet` module), defined in the configuration (`Config.ENCRYPTION_KEY`). Initially, Werkzeug's password hashing (`generate_password_hash`) was used, which is incompatible with the application's Fernet-based verification. The login route expected passwords to be encrypted and decrypted via Fernet, not hashed. This caused the error:
```
Error decrypting password:
Password check failed for user amine
```
due to the decryption attempt:
```python
user.fernet.decrypt(user.password_hash.encode())
```

### Correct Approach (the Explicit and Final Method Clearly)
To create a user correctly, encrypt the password using Fernet:
- Import and instantiate Fernet:
```python
from cryptography.fernet import Fernet
from config import Config
fernet = Fernet(Config.ENCRYPTION_KEY)
```
- Encrypt the password:
```python
encrypted_password = fernet.encrypt('amine'.encode()).decode()
```
- Insert or update user directly into SQLite:
```python
import sqlite3
conn = sqlite3.connect('app/database/database.db')
cursor = conn.cursor()
cursor.execute("""
INSERT INTO users (username, password_hash, employee_id, is_active, role)
VALUES (?, ?, ?, ?, ?)
""", ('amine', encrypted_password, None, 1, 'admin'))
conn.commit()
conn.close()
```
- Or update an existing user:
```python
cursor.execute("UPDATE users SET password_hash=? WHERE username=?", (encrypted_password, 'amine'))
```

### Why This Solved the Issue Explicitly and Clearly
The application uses Fernet encryption, not Werkzeug hashing. Switching to Fernet encryption resolved the login failures, ensuring the stored password matches the expected encryption method.

### Final Result
The encrypted password now matches the application's login logic. Login details:
- URL: `http://localhost:5000/login`
- Username: `amine`
- Password: `amine`
Login works successfully.

## 🔍 Initial Problem
When creating employee (amine2) through the web application, the employee record was created successfully in the employees table, but the user account creation failed, displaying:
```
"Employee created, but user account creation failed, suppressing employee"
```
This occurred because the application couldn’t correctly associate the employee with the user account, leaving `employee_id` as `None` in the users table.

### Steps Extracted from Photo (Manual Database Linking)
1. Open SQLite database:
```bash
sqlite3 app/database/database.db
```
2. Check current user data:
```sql
SELECT id, username, employee_id FROM users WHERE username = 'amine2';
```
3. Identify the employee ID for amine2 from the employees table:
```sql
SELECT id FROM employees WHERE name = 'amine2';
```
(e.g., ID 103)
4. Update the users table:
```sql
UPDATE users SET employee_id = 103 WHERE username = 'amine2';
```
5. Verify the update:
```sql
SELECT id, username, employee_id FROM users WHERE username = 'amine2';
```
6. Test login and shift visibility for amine2.

### Final Result
The problem was solved by manually linking the `employee_id` in the database. Now, user `amine2` can log in and see shifts correctly associated with their employee record.

## 🌐 Host Flask Application
### Host Flask Application on External IP
#### Purpose
The purpose of this task is to make the Flask-based CGSS Shift Management application accessible externally via a public IP (e.g., GCP instance IP) for testing, demonstration, and collaboration. This step ensures administrators and users can interact with the app remotely, supporting the project's scalability and accessibility goals.

#### Steps to Host Flask on an External IP
1. Cloned the Repository
```bash
git clone https://github.com/amine-souissi0/internship2025.git flask-app
cd flask-app
```
2. Created and Activated a Virtual Environment
```bash
python3 -m venv venv
source venv/bin/activate
```
3. Installed Dependencies
```bash
pip install -r requirements.txt
```
4. Handled Port Conflict
```bash
sudo lsof -i :5000
sudo kill -9 80961 80964
```
5. Started the Flask App
```bash
python run.py
```
The app became accessible at `http://0.0.0.0:5000`, enabling external access.

6. Accessed the App via External IP
The GCP instance has the public IP: `34.16.33.223`. By running Flask on `0.0.0.0`, the app was available on all network interfaces. Accessed the app in a browser at: `http://34.16.33.223:5000`.

#### Steps to Access and Work on the Flask Project
##### Purpose
This section provides instructions for other users (e.g., team members or supervisors) to access the GCP instance and work on the Flask project, ensuring seamless collaboration and development.

##### Option 1: SSH into the Instance
- Ask the User to Open Google Cloud Console
  - Navigate to `https://console.cloud.google.com`.
  - Go to Compute Engine → VM Instances.
  - Select the instance named `cgss-dev`.
- Connect via SSH
  - Click the "SSH" button to open a terminal in the browser.
  - Alternatively, use the local terminal with:
```bash
gcloud compute ssh aminisouissi@cgss-dev --zone=us-central1-c
```
(Replace `aminisouissi` with the appropriate username if different.)

##### Option 2: Manually Start the Flask App (If Needed)
- Navigate to the Flask Project Folder
```bash
cd ~/flask-app
```
- Activate the Virtual Environment
```bash
source venv/bin/activate
```
- Run the Flask App
```bash
python run.py
```
This restarts the Flask app, making it accessible again on the external IP (`34.16.33.223:5000`).

#### Conclusion
Hosting the Flask app on an external IP (`34.16.33.223:5000`) enables remote access for administrators and users, aligning with the project's goal of a scalable, cloud-native solution. The provided steps ensure other users can connect to the GCP instance and resume development or testing efficiently.

## 📄 Excel/CSV Upload & Editing
### Excel/CSV File Upload and Editing Feature
#### Purpose
This task adds a feature to the CGSS Shift Management app that lets users upload Excel or CSV files, edit them directly in the browser as a table, and download the updated file. This makes managing shift data easier and more flexible.

#### What You Implemented Step-by-Step
1. **Backend: excel.py**
   - Created a Flask Blueprint called `excel` to handle file uploads.
   - Allowed uploads of `.csv`, `.xlsx`, or `.xls` files.
   - Used pandas to clean the data:
     - Skipped the first row if it’s not needed.
     - Removed empty rows and cells.
     - Fixed column names by removing "Unnamed" or numeric labels.
   - Displayed the cleaned data as an HTML table.
   - Saved the file in the `/uploads` folder.
   - Added a download option at `/excel/download/<filename>`.

2. **Frontend: excel.html (Jinja2 Template)**
   - Added a simple upload form for files.
   - Showed the uploaded data as an editable HTML table.
   - Made all table cells editable with:
```javascript
cell.setAttribute("contenteditable", "true");
```
   - Included a "Download Edited CSV" button that:
     - Collects changes from the table.
     - Creates a new `.csv` file.
     - Downloads it automatically.

#### How to Use It
- Go to the `/excel/` page in your browser.
- Upload a `.csv` or `.xlsx` file.
- The file will appear as a table you can edit by clicking and typing in any cell.
- Click "Download Edited CSV" to save your changes as a new file.
- Use "Download Original File" to get the unchanged version if needed.

#### How We Calculated Shift Statistics – Step by Step
1. **Upload and Read the File**
   - Used pandas to read the uploaded file, starting from the second row as the header:
```python
df = pd.read_excel(file_path, header=[1])
```
   - Stored the data in `parsed_data` for calculations.

2. **Define Shift Types**
   - Set up shift types to track, like "Morning" or "Night".

3. **Check Each Employee**
   - Looped through each row to find employee names:
```python
for _, row in df.iterrows():
    user = row.get("Full Name")
```

4. **Start Counting Shifts**
   - If an employee isn’t in the count list, added them with zero counts for each shift type:
```python
if user not in stats_dict:
    stats_dict[user] = {key: 0 for key in shift_keywords}
```

5. **Count Shifts Per Day**
   - Checked each day’s column for shift types and added to the count:
```python
for col in row.index:
    shift = str(row[col]).strip()
    if shift in stats_dict[user]:
        stats_dict[user][shift] += 1
```

6. **Show the Results**
   - Displayed the counts in `shift_stats.html` as a table with clickable numbers linking to shift details.

![Excel Screenshot](docs/excel%20.png)
![Stats Screenshot](docs/excel%20stats.png)



#### Goal
Make the shift details page show the full context: day, week number, and date.

#### Step-by-Step Explanation
1. **Understand the File Structure**
   - Noted the Excel layout:
     - Row 0: Week info (e.g., Week 39).
     - Row 1: Day names (e.g., Monday).
     - Row 2: Dates (e.g., 25/11).
     - Row 3+: Shift data.

2. **Find the Employee’s Row**
   - When a manager clicks a shift count (e.g., "Morning: 5"), found that employee’s row and checked for matching shifts.

3. **Get the Full Context**
   - For each matching shift, collected:
     - Day from Row 1.
     - Week from Row 0.
     - Date from Row 2.

4. **Format the Information**
   - Combined the data into a clear sentence:
```python
full_display = f"{day_str} - {week_str} - {formatted_date}"
```
   - Example: Monday - Week 39 - 25/11.

5. **Display on the Page**
   - Sent the formatted text to the template:
```python
shifts.append({
    "datetime": full_display,
    "type": shift
})
```
   - Showed it as:
```
Date & Day                | Shift Type
---------------------------|-----------
Monday - Week 39 - 25/11  | Morning
```

#### Why This Is Helpful
- Managers can quickly see the day, week, and date of each shift.
- It simplifies tracking and reporting shift patterns.
- The layout is clean and easy to read.

#### Changes Made
- Updated `shifts.py` with shift time ranges (e.g., "Morning": ["08:00", "16:00"]).
- Added JavaScript to `shift_form.html` to auto-fill time fields based on shift selection.
- Tested to confirm times update correctly.

## ⏰ Hide Start and End Times
### Hide Start and End Times for Employee Shifts
#### Purpose
This task hides the Start Time and End Time columns in the shift management interface to make it simpler and focus on key shift details, improving the user experience.

#### Key Changes
- Hid the Start Time and End Time columns by adding `style="display:none;"` to the table headers (`<th>`) and cells (`<td>`).
- Ensured the hidden data doesn’t affect how shifts work.

#### How to Use It
- Go to the shift management page (e.g., `/shifts/`).
- The table will show shift details without Start Time or End Time columns, making it cleaner and easier to read.

#### Changes Made
- Edited `shifts.html` to add `style="display:none;"` to the Start Time and End Time columns.
- Checked that hiding the columns didn’t break any features.
- Tested the page to ensure it looks good and works as expected.

## 🖱 Editable Board with Dropdowns
### Make Board Editable with Dropdowns and Allow Shift Updates
#### Purpose
The purpose of this task is to make the shift board editable by implementing dropdown menus for shift assignments and enabling real-time updates to shift data, improving flexibility for administrators.

#### What You Implemented Step-by-Step
1. **Backend: shifts.py**
   - Added a route `/update_shift` to handle AJAX requests for updating shift assignments.
   - Implemented logic to update the database with the new shift selection using SQLAlchemy.

2. **Frontend: shifts.html (Jinja2 Template)**
   - Replaced static shift text with a `<select>` dropdown for each shift cell.
   - Populated the dropdown with available shift options (e.g., "Morning", "Night", "Off") using a Jinja2 loop.
   - Added an `onchange` event to trigger an AJAX call to `/update_shift` when a new option is selected.
   - Included JavaScript to send the selected shift value and employee/date context to the backend.

3. **JavaScript**
   - Wrote a function to handle the dropdown change event:
```javascript
function updateShift(selectElement) {
    const shiftId = selectElement.value;
    const employeeId = selectElement.dataset.employeeId;
    const date = selectElement.dataset.date;
    fetch('/update_shift', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ shift_id: shiftId, employee_id: employeeId, date: date })
    }).then(response => response.json())
      .then(data => {
        if (data.success) alert('Shift updated successfully!');
      });
}
```
   - Ensured the dropdown updates the UI dynamically upon successful backend response.

#### How to Use It
- Navigate to the shift board page (e.g., `/shifts/`).
- Click on a shift cell to reveal a dropdown menu with shift options.
- Select a new shift from the dropdown; the change will be saved via AJAX and reflected in the database.
- Verify the update with a success message or refreshed table data.

#### Changes Made
- Modified `shifts.html` to include `<select>` elements with `onchange` events.
- Added `/update_shift` route in `shifts.py` with SQLAlchemy update logic.
- Implemented JavaScript for AJAX calls to handle updates.
- Tested the feature to ensure dropdown selections update shifts correctly and reflect in the database.

## 🐳 Dockerize the Project
### Dockerize the CGSS Shift Management Project
#### Purpose
The purpose of this task is to containerize the CGSS Shift Management project using Docker, enabling consistent deployment across different environments, and to push the resulting image to Docker Hub for broader accessibility.

#### What You Implemented Step-by-Step
1. **Dockerizing the Project**
   - Created a `Dockerfile` to package the project into a Docker image.
   - Built the Docker image with the name `my-flask-app`.
   - Ran the container from the image, with the app successfully running at `http://localhost:5000`.

2. **Steps Followed**
   - Built the Image:
```bash
docker build -t my-flask-app .
```
   - Ran the Container:
```bash
docker run -p 5000:5000 my-flask-app
```
     - Started the container, named `objective_kare`, with port 5000 exposed to the local machine.
   - Checked if It's Running:
     - Verified the container’s status using `docker ps`.

3. **Tag the Image**
   - Associated the image with the Docker Hub account:
```bash
docker tag my-flask-app gocho123/my-flask-app:latest
```
     to link the image to the repository `gocho123/my-flask-app` with the `latest` tag.

4. **Push the Image to Docker Hub**
```bash
docker push gocho123/my-flask-app:latest
```

5. **Verify the Image on Docker Hub**
   - Logged into Docker Hub and confirmed the image appeared in the repositories.
   - The output from the `docker push` command confirmed successful upload.

6. **Pull and Run the Image on Another Machine**
   - Pull the Image:
```bash
docker pull gocho123/my-flask-app:latest
```
     - On another machine, ran to download the image from Docker Hub.
   - Run the Container:
```bash
docker run -p 5000:5000 gocho123/my-flask-app:latest
```
     - Started the container, making the app accessible at `http://localhost:5000` on the new machine.

#### Conclusion
By following these steps, the CGSS Shift Management application was successfully Dockerized, pushed to Docker Hub, and made accessible on any machine with Docker installed, ensuring portability and ease of deployment.
![docker Screenshot](docs/docker.png)
## 🤖 GitHub Actions Automation
### Set Up Automated Builds with Docker and GitHub Actions
#### Purpose
The purpose of this task is to establish an automated build and deployment pipeline using GitHub Actions and Docker, triggered by code changes in the GitHub repository, to streamline the development and deployment process of the CGSS Shift Management application.

#### What You Implemented Step-by-Step
1. **Set Up GitHub Secrets for Docker Hub**
   - Created two GitHub Secrets in the repository's Settings > Secrets:
     - `DOCKER_USERNAME`: Set to the Docker Hub username (e.g., `gocho123`).
     - `DOCKER_PASSWORD`: Set to a Docker Personal Access Token (PAT) generated from Docker Hub, used as the password for authentication.
   - These secrets enable secure authentication to Docker Hub during the CI/CD pipeline.

2. **Create GitHub Actions Workflow for Docker Build and Push**
   - Defined the `docker-build.yml` workflow file in the `.github/workflows/` directory, triggered by pushes or pull requests to the `main` branch.
   - The workflow includes the following steps:
     - **Checkout Code**: Pulls the latest code from the repository.
     - **Set Up Docker Buildx**: Prepares Docker Buildx for building multi-platform images.
     - **Cache Docker Layers**: Caches Docker layers to optimize build times.
     - **Build Docker Image**: Builds the Docker image using the `Dockerfile` and tags it with the commit SHA and `latest`.
     - **Login to Docker Hub**: Authenticates using the credentials from GitHub Secrets.
     - **Push Docker Image to Docker Hub**: Pushes the image with the commit SHA and `latest` tag to the `gocho123/my-flask-app` repository.

3. **Set Up Docker Personal Access Token (PAT)**
   - Generated a Docker PAT from Docker Hub with Public Repo Read-only access permissions.
   - Stored the PAT as the `DOCKER_PASSWORD` secret in GitHub for use in the workflow.

4. **Test the Build and Push Process**
   - Initiated the workflow, resulting in the image being successfully pushed to Docker Hub.
   - The image is now available in the `gocho123/my-flask-app` repository, tagged with both the commit SHA and `latest`.

5. **Verification of Success**
   - Confirmed the image push by visiting the Docker Hub profile, where the `latest` tag was visible.
   - Verified functionality by pulling and running the image locally:
```bash
docker pull gocho123/my-flask-app:latest
docker run -p 5000:5000 gocho123/my-flask-app:latest
```
     to test the Flask app.

6. **Workflow Result**
   - The GitHub Actions workflow ran successfully, with a green checkmark and logs confirming the image was pushed to Docker Hub without errors.

7. **Sequence Code Explanation (Versioning in docker-build.yml)**
   - Added a step named "Get the latest version tag" to handle versioning:
```yaml
- name: Get the latest version tag
  id: get_version
  run: |
    # Get the latest tag from Docker Hub
    latest_tag=$(curl -s https://registry.hub.docker.com/v2/repositories/gocho123/my-flask-app/tags | jq -r '.results | map(.name) | sort | .[-1]')
    echo "Latest tag is: $latest_tag"
    
    # If there is no tag or the tag is "latest", start from version v1.0.0
    if [[ $latest_tag == "latest" || -z "$latest_tag" ]]; then
      new_version="v1.0.0"
    else
      # Split the version into parts and increment the patch version (last part)
      IFS='.' read -r -a version_parts <<< "$latest_tag"
      patch=${version_parts[2]}
      patch=$((patch+1))  # Increment the patch version
      new_version="${version_parts[0]}.${version_parts[1]}.$patch"  # Construct the new version
    fi
    
    echo "New version tag is: $new_version"
    echo "::set-output name=version::$new_version"  # Set the new version as output
```
   - This code retrieves the latest tag, increments the patch version (e.g., from `v1.0.0` to `v1.0.1`), and sets the new version for tagging.

#### Conclusion
The automated build and push process using GitHub Actions ensures that every code change triggers a new Docker image build and deployment to Docker Hub, improving development efficiency and version control.

## ⚙️ Jenkins Automation
### Automate Docker Image Build and Push with Jenkins
#### Purpose
The purpose of this task is to set up a Jenkins pipeline on a Google Cloud VM to automate the process of cloning the GitHub repository, building a Docker image, and pushing it to a local Docker registry, enhancing the CI/CD workflow for the CGSS Shift Management application.

#### What You Implemented Step-by-Step
1. **Created jenkins-fast VM on Google Cloud**
   - Configured the VM with:
     - Region: `us-central1`
     - OS: Debian 12
     - Allowed HTTP/HTTPS traffic
     - Specs: 2 vCPU, 2 GB RAM

2. **Installed Docker and Jenkins**
   - Pulled the official Jenkins Docker image: `jenkins/jenkins:lts`.
   - Mounted the Docker socket to allow Jenkins to access the host Docker.
   - Launched the Jenkins container with:
```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins-with-docker
```

3. **Built Custom Jenkins Image with Docker and Git**
   - Created a `Dockerfile` that installs:
     - `docker.io`
     - `git`
   - Matched the Docker group ID (GID 111) from the host to the Jenkins container.
   - Added the Jenkins user to the Docker group with:
```dockerfile
FROM jenkins/jenkins:lts
USER root
RUN apt-get update && apt-get install -y docker.io git
RUN groupmod -g 111 docker || groupadd -g 111 docker
RUN usermod -aG docker jenkins
USER jenkins
```

4. **Created Jenkins Pipeline Job (build-internship2025)**
   - Configured the pipeline to clone the GitHub repo: `https://github.com/amine-souissi0/internship2025`.
   - Used the following pipeline script:
```groovy
pipeline {
    agent any
    stages {
        stage('Clone GitHub Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/amine-souissi0/internship2025.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t localhost:5000/internship2025 .'
            }
        }
        stage('Push to Local Registry') {
            steps {
                sh 'docker push localhost:5000/internship2025'
            }
        }
    }
}
```

5. **Set Up a Local Docker Registry**
   - Ran a registry container on the host:
```bash
docker run -d -p 5000:5000 --name registry registry:2
```
   - Confirmed it via:
```bash
curl http://localhost:5000/v2/_catalog
```

6. **Final Results**
   - Jenkins now fully automates:
     - Git clone
     - Docker build
     - Docker push to local registry
   - The image is accessible via: `localhost:5000/internship2025`

#### Conclusion
The Jenkins pipeline successfully automates the build and deployment process, integrating with the local Docker registry and ensuring consistent image creation from the GitHub repository.
![Jenkins pipeline successfully automates the build and deployment process Screenshot](docs/jenkins.png)

## 🏗 Terraform Script
### Create a Terraform Script to Provision a Server and Run the Application
#### Purpose
The purpose of this task is to develop a Terraform script to automate the provisioning of a Google Cloud VM, install necessary tools, clone the CGSS Shift Management project, build and run the application using Docker, and make it accessible over the internet, streamlining infrastructure deployment.

#### What is Terraform?
Terraform is an infrastructure-as-code tool that automates the creation and management of cloud resources, allowing the definition of infrastructure in a script rather than manually through a console.

#### What You Implemented Step-by-Step
1. **Created a Folder for the Project**
   - Organized files in a dedicated directory to maintain structure.

2. **Wrote a Terraform Script (main.tf)**
   - Defined a VM configuration in `main.tf` specifying:
     - A VM instance
     - OS: Debian 12
     - A startup script to automate application setup

3. **Startup Script**
   - Embedded a shell script within `main.tf` to:
     - Install `docker.io` and `git`:
```bash
apt-get update && apt-get install -y docker.io git
```
     - Clone the GitHub project:
```bash
git clone https://github.com/amine-souissi0/internship2025.git
```
     - Build the Docker image:
```bash
cd internship2025 && docker build -t my-flask-app .
```
     - Run the Docker container:
```bash
docker run -d -p 5000:5000 my-flask-app
```

4. **Fixed a Permissions Issue**
   - Resolved a permissions error by:
     - Stopping the VM
     - Enabling Compute Engine and Cloud Platform scopes
     - Restarting the VM

5. **Ran Terraform**
   - Executed the following commands to apply the configuration:
```bash
terraform init
terraform apply
```
     - This created the VM and ran the application inside it.

6. **Allowed Traffic on Port 5000**
   - Created a firewall rule to open port 5000 to the internet, enabling external access.

7. **Tested the App**
   - Accessed the application at `http://[your-external-ip]:5000/login` and confirmed it worked.

#### Conclusion
The Terraform script successfully provisions a VM, installs dependencies, builds and runs the CGSS Shift Management application via Docker, and makes it accessible online, demonstrating an automated infrastructure setup.

## 🏖 Off Days Management
### Implement Off Days Management Feature for CGSS Shift Management
#### Purpose
The purpose of this task is to enhance the CGSS Shift Management application by adding a new page to view and manage off days requested by each user. The feature includes tracking an initial allocation of 20 off days per year per user, reducing the available off days upon approval of a request, and clearly displaying the remaining off days for each user.

#### What You Implemented Step-by-Step
1. **Created a New Route (employees.py)**
   - Added a new route `/off_days` to display a page showing each employee’s off-day information.
   - Implemented the `off_days()` function to:
     - Gather data from the database using services.
     - Calculate approved off days and remaining off days for each employee (initially 20 off days per year, reduced by approved requests).
     - Pass the results to the `off_days.html` template.
   - This route ensures that accessing `/off_days` triggers the function to fetch and process the required data dynamically.

2. **Created a Service to Calculate Approved Off Days (employee_shift_service.py)**
   - Implemented the `get_approved_off_days(employee_id)` function to:
     - Query the `employee_shift` table in the database to count approved off days for a specific employee.
     - Used an SQL query to filter rows where `shift_type='OFF'` and `approval_status='Approved'`.
   - This function provides an accurate count of approved off days, which is used to calculate the remaining off days (20 - approved off days).

3. **Created HTML Template (off_days.html)**
   - Designed a Jinja2 template to visualize the off-day data:
     - Iterated over the `off_days_data` passed from the route.
     - Displayed a table with columns for each employee’s name, approved off days, and remaining off days.
   - The template dynamically populates the table, ensuring a clear and user-friendly presentation of the data.

4. **Added Link to Navigation (sidebar.html)**
   - Updated the `sidebar.html` template to include a new `<a>` link directing users to the `/off_days` route using Flask’s `url_for` method.
   - This addition integrates the new feature into the application’s navigation, making it easily accessible to users.

#### How to Use It
- Log in as an admin or user and navigate to the "Off Days" link in the sidebar.
- The `/off_days` page will display a table listing all employees with their approved off days and remaining off days (calculated as 20 minus approved off days).
- Admins can approve off-day requests, which will automatically update the remaining off days for the respective employee.

#### Changes Made
- Added the `/off_days` route in `employees.py` with logic to fetch and calculate off-day data.
- Implemented `get_approved_off_days` in `employee_shift_service.py` to query approved off days.
- Created `off_days.html` to display the off-day information in a table format.
- Updated `sidebar.html` to include a link to the new off-days page.

#### Conclusion
The off days management feature successfully integrates into the CGSS Shift Management application, providing a clear view of off-day allocations and updates, enhancing administrative oversight and user experience.
![offdays Screenshot](docs/off%20days.png)



## ☁️ Docker on GCP with Terraform
### Docker App Deployment on GCP with Terraform for CGSS Shift Management
#### Purpose
The purpose of this task is to deploy the CGSS Shift Management application, packaged as the Docker image `gocho123/my-flask-app:latest`, to a Google Cloud VM using Terraform. The deployment exposes the application on port 5000 and ensures accessibility via a public IP, facilitating scalable and automated deployment.

#### What You Implemented Step-by-Step
1. **Terraform Provisioning**
   - Created a GCP VM named `internship2025-pull` using a Terraform script.
   - Configured a `google_compute_instance` resource and a `google_compute_firewall` rule to allow ingress traffic on TCP port 5000, applying target tags for security.

2. **Firewall Configuration**
   - Updated firewall policies in the GCP Console to remove "Apply to all" and apply target tags (e.g., `allow-docker-5000`) instead, ensuring precise traffic control.

3. **Docker Image Preparation**
   - Built and pushed the updated image `gocho123/my-flask-app:latest` to Docker Hub.
   - Logged into Docker Hub from the VM using a personal access token for authentication.

4. **Deployment on the VM**
   - Pulled the image with:
```bash
docker pull gocho123/my-flask-app:latest
```
   - Ran the container, mapping the internal Flask port 5000 to the VM's port 5000:
```bash
docker run -d -p 5000:5000 --name my-flask-app gocho123/my-flask-app:latest
```

5. **Debugging Port Conflict**
   - Resolved a "port already allocated" error by stopping and removing the old container:
```bash
docker stop my-flask-app
docker rm my-flask-app
```
   - Redeployed the new image after freeing the port, ensuring successful startup.

6. **Verification**
   - Confirmed the Flask app was running using:
```bash
curl http://localhost:5000
```
   - Verified the redirect to `/login`, confirming the backend functionality.

#### How to Use It
- Access the deployed application at `http://34.135.178.175:5000` to verify its operation in the browser.

#### Changes Made
- Wrote a Terraform script to provision the VM and configure the firewall.
- Updated firewall rules in GCP to allow port 5000 traffic.
- Pulled and ran the Docker image on the VM, resolving port conflicts.
- Tested the deployment to ensure accessibility.

#### Next Steps
- Ensure the app logic works as expected at `http://34.135.178.175:5000`.
- Optionally automate redeployment using a shell script or CI/CD pipeline.

#### Conclusion
The Terraform-based deployment successfully provisions a VM, deploys the CGSS Shift Management application via Docker, and makes it accessible, aligning with the project's goal of scalable deployment.

## 🔑 SSH Authentication
### SSH Authentication Setup for VM Access in CGSS Shift Management
#### Purpose
The purpose of this task is to implement SSH key-based authentication for secure, passwordless access to the Google Cloud VM hosting the CGSS Shift Management application, enhancing security and simplifying access control for team members.

#### Why It’s Important
- **Security**: SSH keys are more secure than passwords, reducing the risk of unauthorized access.
- **Speed**: Instant login without password entry improves efficiency.
- **Access Control**: Individual keys allow easy management of user access.
- **Automation**: Supports secure script-based connections to the VM.

#### What You Implemented Step-by-Step
1. **SSH Key Pair Creation**
   - Generated a secure SSH key pair (ed25519) locally using:
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
   - Copied the public key from `~/.ssh/id_ed25519.pub`.

2. **VM Configuration**
   - Added the public key to the VM’s `~/.ssh/authorized_keys` file.
   - Set proper permissions:
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

3. **Tested the Connection**
   - Tested the login using:
```bash
ssh -i ~/.ssh/id_ed25519 username@VM_IP
```
   - Confirmed successful login without a password.

#### How to Use It
- Team members can use their SSH private key to access the VM securely by running the `ssh` command with the appropriate key and VM IP.

#### Changes Made
- Generated an ed25519 SSH key pair locally.
- Added the public key to the VM’s `authorized_keys` file.
- Configured permissions and tested the connection.

#### Conclusion
SSH key-based authentication ensures secure and efficient access to the VM hosting the CGSS Shift Management application, aligning with best practices for secure VM management.

## 📝 Deployment Summary
### Deployment Summary of Flask Application on VM using Docker for CGSS Shift Management
#### Purpose
The purpose of this task is to deploy the CGSS Shift Management Flask application on a VM using Docker, ensuring it runs efficiently and is accessible via a public IP, replacing any conflicting services and optimizing the deployment process.

#### What You Implemented Step-by-Step
1. **VM Preparation**
   - Connected to the VM via SSH using its public IP address.
   - Verified Docker installation with `docker --version`.
   - Ensured Docker was configured and running.

2. **Managing Existing Docker Containers**
   - Listed all containers with `docker ps -a`.
   - Identified an existing `my-flask-app` container and a `guacamole` service using port 80.
   - Stopped the `guacamole` container to free port 80:
```bash
docker stop guacamole
```
   - Stopped and removed the old `my-flask-app` container:
```bash
docker stop my-flask-app
docker rm my-flask-app
```

3. **Downloading and Running the Flask Container**
   - Pulled the latest image from Docker Hub:
```bash
docker pull gocho123/my-flask-app:latest
```
   - Started the container, mapping port 5000 to 80:
```bash
docker run -d -p 80:5000 --name my-flask-app gocho123/my-flask-app:latest
```
   - Verified the container status with `docker ps`.

4. **Network Configuration and External Access**
   - Confirmed the VM’s public IP as `35.238.185.239`.
   - Ensured GCP firewall rules allowed inbound traffic on port 80.
   - Accessed the application at `http://35.238.185.239/`.

#### Changes Made
- Stopped and removed conflicting Docker containers.
- Pulled and ran the updated Flask image on port 80.
- Configured network settings for public access.

#### Conclusion
The Flask application is successfully deployed on the VM using Docker, accessible at `http://35.238.185.239/`, with conflicts resolved and optimal port usage ensured.

## ☸️ Kubernetes Cluster Setup
### Implement Kubernetes Cluster Setup for CGSS Shift Management
#### Purpose
The purpose of this task was to create a Kubernetes cluster on a bare-metal machine using QEMU/KVM virtual machines (VMs) to deploy and manage the CGSS Shift Management Flask application. This setup, involving a control plane (master) node and a worker node, was a multi-day effort to ensure scalability and resilience, integrating with previous deployment strategies.

#### Goal
- Create a Kubernetes cluster with:
  - 1 Control Plane node (Master): `cgss-master`
  - 1 Worker node: `amine`
- Deploy the Flask application within this cluster for enhanced management and scalability.

#### Tools Used
| Tool            | Purpose                                              |
|-----------------|------------------------------------------------------|
| QEMU/KVM        | Creates and manages virtual machines                 |
| virt-install    | Simplifies VM creation with libvirt                  |
| Ubuntu 22.04    | Operating system for VMs                            |
| kubeadm         | Initializes and configures Kubernetes cluster        |
| kubelet, kubectl| Core Kubernetes components for node management and CLI operations |

#### Step-by-Step Instructions
**✅ Step 1: Prepare the Host Machine**
- **Objective**: Ensure the physical host machine (running Arch Linux) is ready for virtualization.
- **Requirements**: At least 16 CPU cores and 32 GB RAM.
- **Actions**:
  - Install virtualization tools:
```bash
sudo pacman -S qemu virt-manager bridge-utils libvirt
```
  - Enable and start libvirt service:
```bash
sudo systemctl enable --now libvirtd
```
- **Time Investment**: 1 day to resolve initial networking issues with the default virtual bridge.
- **Verification**: Confirmed with `sudo systemctl status libvirtd`.

**✅ Step 2: Create the Master Node VM (cgss-master)**
- **Objective**: Set up the control plane node with adequate resources.
- **Actions**:
  - Create a 20GB disk image:
```bash
qemu-img create -f qcow2 /var/lib/libvirt/images/cgss-master.qcow2 20G
```
  - Install Ubuntu 22.04 VM:
```bash
virt-install \
  --name cgss-master \
  --memory 4096 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/cgss-master.qcow2,format=qcow2 \
  --network network=default \
  --cdrom /var/lib/libvirt/boot/ubuntu-22.04.3-live-server-amd64.iso \
  --os-variant ubuntu22.04
```
- **Time Investment**: 2 days due to initial networking misconfigurations, resolved by adjusting the virtual bridge settings and ensuring proper IP assignments.
- **Verification**: Successfully logged into the VM via `virsh console cgss-master` and confirmed Ubuntu 22.04 installation with `lsb_release -a`.

**✅ Step 3: Create the Worker Node VM (amine)**
- **Objective**: Establish a worker node to handle application workloads and ensure cluster scalability.
- **Actions**:
  - Create a 20GB disk image:
```bash
qemu-img create -f qcow2 /var/lib/libvirt/images/amine.qcow2 20G
```
  - Install Ubuntu 22.04 VM:
```bash
virt-install \
  --name amine \
  --memory 4096 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/amine.qcow2,format=qcow2 \
  --network network=default \
  --cdrom /var/lib/libvirt/boot/ubuntu-22.04.3-live-server-amd64.iso \
  --os-variant ubuntu22.04
```
- **Time Investment**: 1 day, optimized by reusing the master node setup process.
- **Verification**: Accessed the VM via `virsh console amine` and verified Ubuntu installation with `lsb_release -a`.

**✅ Step 4: Configure Networking Between Nodes**
- **Objective**: Ensure seamless communication between the master and worker nodes for cluster operations.
- **Actions**:
  - Assigned static IPs within the default virtual network (`192.168.122.10` for `cgss-master`, `192.168.122.11` for `amine`).
  - Edited `/etc/netplan/01-netcfg.yaml` on both VMs:
```yaml
network:
  version: 2
  ethernets:
    ens3:
      addresses: [192.168.122.10/24]
      gateway4: 192.168.122.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```
  - Applied the configuration:
```bash
sudo netplan apply
```
  - Tested connectivity with `ping 192.168.122.11` from the master and confirmed bidirectional communication.
- **Time Investment**: 1 day to resolve initial firewall and bridge configuration issues.
- **Verification**: Confirmed successful ping and established SSH access between nodes.

**✅ Step 5: Install Kubernetes Components**
- **Objective**: Prepare both nodes with necessary Kubernetes tools for cluster management.
- **Actions**:
  - Updated system packages and installed dependencies:
```bash
sudo apt update && sudo apt install -y apt-transport-https curl
```
  - Added the Kubernetes repository and GPG key:
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
  - Installed Kubernetes tools:
```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
```
  - Disabled swap to comply with Kubernetes requirements:
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```
- **Time Investment**: 1 day to troubleshoot package installation conflicts.
- **Verification**: Validated installations with `kubeadm version` and `kubectl version`.

**✅ Step 6: Initialize the Kubernetes Cluster**
- **Objective**: Configure the master node as the control plane and establish the cluster network.
- **Actions**:
  - Initialized the cluster on the master node:
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.122.10
```
  - Initially installed Flannel as the pod network, but faced connectivity issues:
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
  - Switched to Calico to resolve network instability and enhance performance:
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
  - Configured kubectl for the root user:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
- **Time Investment**: 2 days to address network plugin issues, with Calico providing a more stable solution.
- **Verification**: Checked cluster status with `kubectl get nodes` and confirmed Calico pods were operational.

**✅ Step 7: Join the Worker Node**
- **Objective**: Integrate the worker node into the Kubernetes cluster.
- **Actions**:
  - Retrieved the join command from the master’s `kubeadm init` output (e.g., `kubeadm join 192.168.122.10:6443 --token abcdef.1234567890abcdef --discovery-token-ca-cert-hash sha256:12345...`).
  - Executed the join command on the worker node:
```bash
sudo kubeadm join 192.168.122.10:6443 --token abcdef.1234567890abcdef --discovery-token-ca-cert-hash sha256:12345...
```
- **Time Investment**: 1 day to resolve token expiration and network configuration errors.
- **Verification**: Confirmed both nodes were Ready with `kubectl get nodes`.

**✅ Step 8: Deploy the Flask Application**
- **Objective**: Deploy the CGSS Shift Management Flask application to the Kubernetes cluster.
- **Actions**:
  - Created a deployment YAML file (`flask-deployment.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image: gocho123/my-flask-app:latest
        ports:
        - containerPort: 5000
```
  - Created a service YAML file (`flask-service.yaml`):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  selector:
    app: flask-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
```
  - Applied the configurations:
```bash
kubectl apply -f flask-deployment.yaml
kubectl apply -f flask-service.yaml
```
- **Time Investment**: 2 days to debug pod scheduling and optimize Calico network settings.
- **Verification**: Accessed the app via the external IP from `kubectl get services` and tested core functionalities as of 11:55 AM +08, Thursday, July 17, 2025.

#### Challenges Encountered
- Initial networking issues with the default bridge required manual adjustments to virtual network settings.
- Flannel’s network instability necessitated a switch to Calico, which provided better performance and troubleshooting capabilities.
- Swap memory conflicts delayed initial Kubernetes setup, resolved by disabling swap permanently.

#### Conclusion
The Kubernetes cluster, comprising a master node (`cgss-master`) and a worker node (`amine`) on QEMU/KVM VMs, successfully hosts the CGSS Shift Management Flask application. The adoption of Calico as the pod network addressed early connectivity issues, ensuring a scalable and resilient deployment as of 11:55 AM +08, Thursday, July 17, 2025.

## 🚀 Deploy TeamCity on Kubernetes
### Deploy TeamCity on Kubernetes for CGSS Shift Management
#### Purpose
The purpose of this task is to deploy TeamCity, a Continuous Integration and Continuous Deployment (CI/CD) server by JetBrains, on the Kubernetes cluster to automate the build, test, and deployment processes for the CGSS Shift Management Flask application. TeamCity enhances development efficiency by automating code compilation, testing, and deployment to various environments, integrating seamlessly with Kubernetes for containerized workflows.

![teamcity Screenshot](docs/teamcity.png)
#### Why TeamCity?
Your manager requested TeamCity to automate deployments, eliminating manual rebuilds and redeployments with each code change. It ensures reliable, repeatable deployments through standardized testing and aligns with Kubernetes usage in your company. As a cornerstone of DevOps, TeamCity boosts agility, reduces risks, and supports team collaboration with centralized build logs and deployment history.

#### Relationship with Kubernetes
- **Automated CI/CD for Kubernetes Workloads**: TeamCity builds Docker images, runs tests, and deploys them to Kubernetes, triggered by code pushes for a streamlined workflow.
- **Deployment Automation**: Build configurations in TeamCity create Docker images, push them to registries like Docker Hub, and update Kubernetes with tools like `kubectl`.
- **Managing Kubernetes Manifests**: TeamCity stores and manages deployment files (e.g., `deployment.yaml`), facilitating version-controlled Kubernetes configurations.
- **Feedback & Quality Gates**: Real-time feedback on builds, tests, and code quality ensures safer Kubernetes deployments with fewer errors.
- **Team Collaboration**: Centralized logs, history, and artifacts enhance team visibility into deployment activities.

#### What You Implemented Step-by-Step
1. **Prepare Your YAML Deployment File**
   - **Objective**: Define the TeamCity deployment configuration.
   - **Actions**:
     - Created a YAML file named `teamcity-deployment.yaml` with the following content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: teamcity-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: teamcity-server
  template:
    metadata:
      labels:
        app: teamcity-server
    spec:
      containers:
      - name: teamcity-server
        image: jetbrains/teamcity-server
        ports:
        - containerPort: 8111
        volumeMounts:
        - name: teamcity-data
          mountPath: /data/teamcity_server/datadir
        - name: teamcity-logs
          mountPath: /opt/teamcity/logs
      volumes:
      - name: teamcity-data
        emptyDir: {}
      - name: teamcity-logs
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: teamcity-server
spec:
  type: NodePort
  selector:
    app: teamcity-server
  ports:
    - port: 8111
      targetPort: 8111
      nodePort: 30111  # Exposes TeamCity on this port on the node
```

2. **Deploy TeamCity to Kubernetes**
   - **Objective**: Deploy the TeamCity server to the cluster.
   - **Actions**:
     - Applied the deployment manifest:
```bash
kubectl apply -f teamcity-deployment.yaml
```
   - **Time Investment**: 1 day to ensure proper application.

3. **Verify Pods and Services**
   - **Objective**: Confirm the deployment is running correctly.
   - **Actions**:
     - Checked pod status:
```bash
kubectl get pods -o wide
```
       - Example output:
```
NAME                                 READY   STATUS    RESTARTS   AGE     IP            NODE
teamcity-server-xxxxxxxxxx-xxxxx      1/1     Running   0          3m      <Pod IP>      <Node>
```
     - Checked service details:
```bash
kubectl get svc teamcity-server
```
       - Example output:
```
NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
teamcity-server   NodePort   10.99.193.243   <none>        8111:30111/TCP    3m
```
   - **Time Investment**: 30 minutes to verify and troubleshoot.

4. **Test Access to TeamCity**
   - **Objective**: Ensure the TeamCity UI is accessible.
   - **Actions**:
     - Tested access from another VM on the same subnet:
```bash
curl http://<NODE_PUBLIC_IP>:30111
```
       or opened `http://<NODE_PUBLIC_IP>:30111` in a web browser.
     - **Troubleshooting**: If not accessible, checked firewall rules to ensure port 30111 was open and reviewed pod logs with:
```bash
kubectl logs <pod-name>
```
   - **Time Investment**: 1 day to resolve firewall and network access issues.

5. **Complete TeamCity Initial Setup**
   - **Objective**: Finalize the TeamCity configuration.
   - **Actions**:
     - Accessed the TeamCity web setup wizard at `http://<NODE_PUBLIC_IP>:30111`.
     - Accepted the license agreement and set up the data directory (used default settings for testing).
     - Created an administrator user account.
   - **Time Investment**: 1 hour to complete the wizard.

6. **Agent Setup**
   - **Objective**: Configure a TeamCity build agent to execute builds.
   - **Actions**:
     - Downloaded the TeamCity Build Agent from the TeamCity server UI.
     - Extracted the agent files on an Ubuntu VM and edited `conf/buildAgent.properties` to set:
```
serverUrl=http://<NODE_IP>:30111
```
     - Started the agent:
```bash
./bin/agent.sh start
```
     - Authorized the agent from the TeamCity web UI under the Agents section.
   - **Time Investment**: 2 hours to download, configure, and authorize.
   - **Verification**: Confirmed the agent was listed as connected in the TeamCity dashboard.

7. **Expose TeamCity UI via SSH Tunnel**
   - **Objective**: Enable local browser access to TeamCity.
   - **Actions**:
     - Set up an SSH tunnel from the laptop to `poha-webserver`:
```bash
ssh -L 8111:localhost:8111 aminisouissi@35.208.94.224
```
     - Forwarded traffic from `poha-webserver` to `cgss-master`:
```bash
ssh -L 8111:192.168.122.10:30111 root@localhost -p 4444
```
     - Accessed TeamCity at `http://localhost:8111`.
   - **Time Investment**: 1 day to configure and test the tunneling setup.

8. **Automate Reverse SSH Tunnel for Flask App**
   - **Objective**: Automate access to the Flask app via a reverse tunnel.
   - **Actions**:
     - Configured `autossh` reverse tunnel on `cgss-master`:
```bash
/usr/bin/autossh -M 0 -N -i /home/apohorai/.ssh/id_ed25519 -R 31111:192.168.122.10:31111 attila_pohorai@35.208.94.224
```
     - Tested with `curl http://localhost:31111` on `cgss-master`.
     - Created `autossh-tunnel.service`:
```
[Unit]
Description=Auto SSH Tunnel for Flask App
After=network.target

[Service]
User=root
ExecStart=/usr/bin/autossh -M 0 -N -i /home/apohorai/.ssh/id_ed25519 -R 31111:192.168.122.10:31111 attila_pohorai@35.208.94.224
Restart=always

[Install]
WantedBy=multi-user.target
```
     - Enabled and started the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable autossh-tunnel.service
sudo systemctl start autossh-tunnel.service
```
     - Verified with `sudo systemctl status autossh-tunnel.service` and after reboot.
   - **Time Investment**: 2 days to troubleshoot and automate.

#### Challenges Encountered
- Initial pod crashes due to insufficient disk space, resolved by cleaning logs and images.
- Firewall restrictions delayed UI access, addressed by adjusting rules and using SSH tunnels.
- Agent authorization required multiple attempts due to network latency with Calico.

#### Conclusion
TeamCity is successfully deployed on the Kubernetes cluster as of 11:55 AM +08, Thursday, July 17, 2025, with a configured agent and accessible UI via SSH tunneling. The automated reverse tunnel ensures consistent Flask app access, providing a robust CI/CD foundation for the CGSS Shift Management project.

## 🛠 Deploy Flask App with TeamCity
### Deploy Flask App Using TeamCity on Kubernetes for CGSS Shift Management
#### Purpose
This task outlines the process of deploying the CGSS Shift Management Flask application using TeamCity’s CI/CD pipeline on the Kubernetes cluster, leveraging the previously configured TeamCity server and agent to automate the build and deployment process.

#### What You Implemented Step-by-Step
1. **Set Up TeamCity Agent**
   - **Objective**: Configure a build agent to execute Flask app builds.
   - **Actions**:
     - Installed and configured a TeamCity build agent on an Ubuntu VM.
     - Ensured the agent was connected and authorized in the TeamCity server dashboard at `http://localhost:8111`.
   - **Time Investment**: 2 hours to install and authorize.
   - **Verification**: Confirmed agent status in the TeamCity Agents section.

2. **Connected to GitHub Repository**
   - **Objective**: Link the Flask app source code to TeamCity.
   - **Actions**:
     - Created a new project in TeamCity using the URL `https://github.com/LemuJr14/adc_webapp`.
     - Generated a GitHub personal access token with repo access permissions.
     - Provided the repository URL, username, and token in the TeamCity setup wizard.
     - Verified the connection with a green check mark in TeamCity.
   - **Time Investment**: 1 hour to configure and verify.

3. **Configured Build Steps**
   - **Objective**: Define the build process for the Flask app.
   - **Actions**:
     - Added a "Command Line" build step to install requirements:
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```
     - Added a second "Command Line" build step to run the app:
```bash
source venv/bin/activate
python run.py
```
   - **Time Investment**: 1 hour to customize steps.

4. **Ran the Build**
   - **Objective**: Execute the build process.
   - **Actions**:
     - Clicked the "Run" button in TeamCity to start the build.
     - Monitored the build log, confirming:
       - Requirements installed successfully.
       - Flask app started with output:
```
* Running on all addresses (0.0.0.0)
* Running on http://127.0.0.1:5000
* Running on http://192.168.122.76:5000
```
   - **Time Investment**: 30 minutes to run and monitor.

5. **Accessed the Running Application**
   - **Objective**: Verify the app is accessible.
   - **Actions**:
     - Accessed the app in a browser at `http://192.168.122.76:5000` via the agent’s IP.
     - Tested core functionalities (e.g., shift selection, request submission) as of 11:55 AM +08, Thursday, July 17, 2025.
   - **Time Investment**: 30 minutes to test access.

#### Notes and Best Practices
- The Flask app uses the built-in development server, suitable for testing but not production.
- For production, use a WSGI server like Gunicorn.
- Ensure port 5000 is open on the VM firewall for external access.

#### Summary Table
| Step             | Description                                                  |
|------------------|--------------------------------------------------------------|
| Agent Setup      | Connected build agent to TeamCity server                     |
| VCS Integration  | Linked GitHub repo with access token                         |
| Build Steps      | 1. Install Python dependencies<br>2. Run Flask app           |
| Build Execution  | Ran build and checked logs for successful startup            |
| App Access       | Opened the app in browser using agent IP and port 5000       |

#### Challenges Encountered
- Initial agent connection issues, resolved by verifying network settings.
- Build failures due to missing dependencies, fixed by updating `requirements.txt`.

#### Conclusion
The Flask application is successfully deployed using TeamCity’s CI/CD pipeline on Kubernetes as of 11:55 AM +08, Thursday, July 17, 2025, with automated builds and accessible functionality, marking a significant step in project automation.

## 🐳 Deploy Flask App with TeamCity & Docker
### Deploy Flask App with Docker Using TeamCity on Kubernetes for CGSS Shift Management
#### Purpose
This task details the deployment of the CGSS Shift Management Flask application using Docker and TeamCity’s CI/CD pipeline on the Kubernetes cluster, ensuring containerized and versioned deployments.

#### What You Implemented Step-by-Step
1. **Prepare Your Project**
   - **Objective**: Set up the project for Dockerization.
   - **Actions**:
     - Created a `Dockerfile` at the project root:
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt
COPY . /app/
EXPOSE 5000
CMD ["python", "run.py"]
```
     - Uploaded the project to the GitHub repository `https://github.com/LemuJr14/adc_webapp`.
   - **Time Investment**: 2 hours to create and upload.

2. **Set Up TeamCity**
   - **Objective**: Configure TeamCity for the Docker build.
   - **Actions**:
     - Connected the GitHub repository to TeamCity as a new project.
     - Ensured the TeamCity build agent had Docker installed and was running.
   - **Time Investment**: 1 hour to configure.

3. **Configure Docker Hub Authentication**
   - **Objective**: Securely authenticate with Docker Hub.
   - **Actions**:
     - Generated a Docker Hub Personal Access Token with Write permissions.
     - Added a "Command Line" build step in TeamCity:
```bash
echo "YOUR_ACCESS_TOKEN" | docker login -u gocho123 --password-stdin
```
   - **Time Investment**: 30 minutes to set up.

4. **Configure Build Steps in TeamCity**
   - **Objective**: Define the Docker build and push process.
   - **Actions**:
     - Added a "Docker Build" step with image name `gocho123/my-flask-app:latest`.
     - Added a "Docker Push" step to push the image to Docker Hub.
   - **Time Investment**: 1 hour to configure steps.

5. **Run the Pipeline**
   - **Objective**: Execute the CI/CD pipeline.
   - **Actions**:
     - Triggered the build in TeamCity.
     - Monitored the build log for errors; on success, confirmed "Success" and image upload to Docker Hub.
   - **Time Investment**: 30 minutes to run and verify.

6. **Verify the Result**
   - **Objective**: Confirm the Docker image deployment.
   - **Actions**:
     - Checked Docker Hub to verify the image `gocho123/my-flask-app:latest` was pushed.
     - Optionally pulled the image on another machine:
```bash
docker pull gocho123/my-flask-app:latest
```
   - **Time Investment**: 30 minutes to verify.

7. **Deploy to Kubernetes**
   - **Objective**: Deploy the Dockerized app to Kubernetes.
   - **Actions**:
     - Updated the `flask-deployment.yaml` to use the latest image:
```yaml
image: gocho123/my-flask-app:latest
```
     - Applied the updated deployment:
```bash
kubectl apply -f flask-deployment.yaml
```
     - Verified pod status with `kubectl get pods`.
   - **Time Investment**: 1 hour to deploy and test.

#### Summary for Your Report
I automated the process of building and pushing a Docker image of my Flask app using TeamCity. After connecting my GitHub repository, I configured TeamCity to build my Docker image and push it to Docker Hub every time a build runs. I used a secure Docker Hub access token for authentication and verified the image was successfully pushed by checking Docker Hub and pulling the image. This setup allows quick deployment anywhere and ensures the image is always up-to-date as of 11:55 AM +08, Thursday, July 17, 2025.

#### Challenges Encountered
- Initial Docker build failures due to missing dependencies, resolved by updating `requirements.txt`.
- Authentication issues with Docker Hub, fixed by correctly configuring the access token.

#### Conclusion
The Flask application is successfully deployed with Docker using TeamCity on Kubernetes as of 11:55 AM +08, Thursday, July 17, 2025, providing a containerized and automated deployment solution for the CGSS Shift Management project.

## ⚙️ Ansible Automation
### Implement Ansible Automation for CGSS Shift Management
#### Purpose
This task uses Ansible to automate the setup and configuration of the CGSS Shift Management application on the Kubernetes cluster, reducing manual effort and ensuring consistency across deployments.

#### What You Implemented Step-by-Step
1. **Set Up Ansible Control Node**
   - **Objective**: Prepare the host for Ansible.
   - **Actions**:
     - Installed Ansible on the control node:
```bash
sudo pacman -S ansible
```
     - Created `inventory.yml` with the IPs of `cgss-master` and `amine`.
   - **Time Investment**: 1 hour to install and configure.

2. **Create Playbook**
   - **Objective**: Automate cluster and app deployment.
   - **Actions**:
     - Developed `deploy.yml` to install dependencies and deploy the app:
```yaml
- hosts: all
  become: yes
  tasks:
    - name: Install Docker
      apt:
        name: docker.io
        state: present
    - name: Install Kubernetes tools
      apt:
        name: "{{ packages }}"
      vars:
        packages:
        - kubelet
        - kubeadm
        - kubectl
    - name: Deploy Flask app
      command: kubectl apply -f /path/to/flask-deployment.yaml
```
   - **Time Investment**: 2 hours to write and test.

3. **Run the Playbook**
   - **Objective**: Execute the automation script.
   - **Actions**:
     - Ran the playbook:
```bash
ansible-playbook -i inventory.yml deploy.yml
```
   - **Time Investment**: 1 hour to execute.

4. **Verification**
   - **Objective**: Confirm automation success.
   - **Actions**:
     - Checked Docker and Kubernetes installations with `docker --version` and `kubectl version`.
     - Verified app deployment with `kubectl get pods`.
   - **Time Investment**: 30 minutes to verify.

#### Challenges Encountered
- Initial playbook errors due to incorrect paths, resolved by updating file references.
- Network delays with Calico required playbook retries.

#### Conclusion
Ansible automation streamlines the CGSS Shift Management deployment, ensuring consistency across nodes as of 11:55 AM +08, Thursday, July 17, 2025.
![ansible Screenshot](docs/pipeline.png)



## 🎉 Conclusion
The CGSS Shift Management project has evolved from an Excel-based system to a cloud-native Flask application deployed on a Kubernetes cluster, with automation via TeamCity and Ansible. As of 11:55 AM +08, Thursday, July 17, 2025, this comprehensive setup, detailed over weeks of effort, ensures scalability, security, and efficiency, marking a significant advancement in modern DevOps practices for the project.
