# Student Registration System

A simple **Flask** web application to manage student records with **MongoDB** as the backend database. Users can **add, view, update, and delete** student details.

---

## Features

* List all students on the home page
* Add a new student
* Update existing student details
* Delete a student with confirmation
* Simple and responsive UI using Bootstrap

---

## Tech Stack

* **Backend:** Python, Flask
* **Database:** MongoDB (via Flask-PyMongo)
* **Frontend:** HTML, Jinja2 templates, Bootstrap 5
* **Environment Variables:** Managed via `.env` file

---

## Setup Instructions

### 1. Clone the repository

```bash
git clone <your-repo-url>
cd <repo-folder>
```

### 2. Create and activate a virtual environment

```bash
python -m venv venv
# Activate venv
# Windows:
venv\Scripts\activate
# Linux / Mac:
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

**`requirements.txt` example:**

```
Flask
Flask-PyMongo
python-dotenv
bson
```

### 4. Configure environment variables

Create a `.env` file in the project root:

```
MONGO_URI=<your-mongodb-connection-string>
SECRET_KEY=<your-secret-key>
```

### 5. Run the application

```bash
python app.py
```

Open your browser at: [http://localhost:5000](http://localhost:5000)

---

## Project Structure

```
project/
│
├── templates/
│   ├── base.html
│   ├── index.html
│   ├── add_student.html
│   ├── update_student.html
│
├── app.py
├── requirements.txt
└── .env
```

---

## Screenshots

**Home Page**
Lists all students with Edit/Delete buttons.
- <img width="1902" height="607" alt="image" src="https://github.com/user-attachments/assets/a58a6a6d-4978-4769-8074-232e4d31e69d" />


**Add Student**
Form to add a new student.
- <img width="1897" height="801" alt="image" src="https://github.com/user-attachments/assets/d65d25c3-ebb5-410a-adb1-e130ad7c5878" />


**Update Student**
Form pre-filled with student details.
- <img width="1905" height="897" alt="image" src="https://github.com/user-attachments/assets/04febf01-879f-431f-ab07-abcfb993acf1" />



---

## Jenkins CI/CD Pipeline

This repository includes a Jenkins pipeline for running a build, unit tests, and a staging deployment whenever code is pushed to the `main` branch.

### Pipeline stages

1. Build
   - Creates a Python virtual environment
   - Upgrades `pip`
   - Installs dependencies from `requirements.txt`

2. Test
   - Runs `pytest -q` against the project tests

3. Deploy
   - Copies the project into `/opt/flask_practice`
   - Starts the Flask application in the background using `nohup`
   - Runs only on the `main` branch

### Prerequisites

- Jenkins installed on a machine or VM
- Java runtime installed with Jenkins
- Python 3 and `pip` available on the Jenkins agent
- Git installed
- GitHub repository connected to Jenkins
- Required Jenkins plugins:
  - GitHub Integration / GitHub plugin
  - Pipeline
  - Email Extension plugin (optional but required for email notifications)
- MongoDB running and accessible via the app's `.env` configuration

### Jenkins setup steps

1. Install Jenkins and open the Jenkins dashboard.
2. Install the required plugins.
3. Create a new pipeline job.
4. Point the pipeline to this repository.
5. Use the repository's `Jenkinsfile`.
6. Enable GitHub hook or configure polling for the `main` branch.
7. Save the job and trigger a build.

### GitHub trigger configuration

To trigger builds when changes are pushed to `main`, configure either:

- a GitHub webhook, or
- Jenkins polling for the repository branch

The `Jenkinsfile` includes:

```groovy
triggers {
    githubPush()
}
```

This works when GitHub integration is enabled and the repo is linked to Jenkins.

### Email notifications

The pipeline uses the Jenkins `mail` step to send build status notifications on success and failure.

Update the recipient email in the `Jenkinsfile` before running it in production:

```groovy
mail to: 'ankit@example.com'
```

### Example usage

```bash
git clone <your-forked-repo-url>
cd flask_Practice
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pytest -q
python app.py
```

### Notes

* Make sure MongoDB is running and accessible via the URI in `.env`
* Delete action includes a confirmation page to prevent accidental deletion
* Uses `ObjectId` from `bson` to work with MongoDB document IDs
* If you use MongoDB Atlas on macOS, install dependencies again (`pip install -r requirements.txt`). This project now uses `certifi` CA bundle explicitly to avoid common TLS certificate verification failures with `pymongo`.
* For production deployments, replace the simple `nohup` staging deploy step with a proper systemd service, Docker deployment, or another deployment strategy.

---

## License

MIT License

---



