# PR Analysis Coverage Pipeline

This repository contains a **Jenkins Declarative Pipeline** that runs unit tests on a Java/Maven application, generates **JaCoCo code coverage reports**, and posts a coverage summary directly to the corresponding **Bitbucket Pull Request (PR)**.

Unlike traditional coverage checks, this pipeline focuses only on the **changed lines of code in the PR**, ensuring that developers maintain or improve test coverage in their contributions.

---

## 🚀 Features

* **Automated Checkout**
  Clones the source code and PR branch directly from Bitbucket SCM.

* **Test Execution in Docker**
  Uses a containerized `maven:3.6.3-openjdk-8` environment to run `mvn clean test`.

* **Coverage Reporting**
  Generates **JaCoCo XML reports** under `target/site/jacoco/jacoco.xml`.

* **Diff-based Coverage Analysis**
  Uses [`diff-cover`](https://github.com/Bachmann1234/diff_cover) (Python) to compare coverage against only the changed lines in the PR.
  Produces both:

  * **Raw text summary** (inline in Jenkins logs)
  * **HTML report** (`coverage_report.html`)

* **Bitbucket Integration**

  * Posts **build status checks** (`SUCCESSFUL` or `FAILED`) on the commit.
  * Adds a **detailed coverage comment** to the PR with:

    * Coverage % for changed lines
    * Number of changed files
    * Number of Java files changed
    * Count of uncovered lines
    * Top files missing coverage

* **Threshold Enforcement**
  Shows the build status as failed if  coverage falls below a configurable threshold (default: **50%**).

---

## 🛠️ Requirements

* **Jenkins** (Pipeline + Docker agents enabled)
* **Bitbucket credentials** stored in Jenkins:

  * ID: `Whatever you create`
  * Type: `Username with password`
* **Maven** project with JaCoCo plugin configured (`jacoco.xml` must be generated under `target/site/jacoco/`).

---

## ⚙️ Configuration

Before using this pipeline, update the following placeholders in `Jenkinsfile`:

* **Bitbucket Credentials**

  ```groovy
  withCredentials([usernamePassword(credentialsId: 'bitbucket_creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')])
  ```

* **Repository Information**
  Update your **Bitbucket workspace** and **repository name** in API calls:

  ```bash
  https://api.bitbucket.org/2.0/repositories/<workspace>/<repo>/
  ```

* **Project Directory Path**
  Replace `<Directory-path>` with the directory containing your Maven module (where `target/site/jacoco/jacoco.xml` is generated).

* **Coverage Threshold**
  Adjust threshold in Groovy:

  ```groovy
  def threshold = 50.0
  ```

---

## 📊 Example Output

When a PR is raised, the pipeline will post a comment like:

```
#### 📊 Code Coverage Report for PR #42
**Branch:** feature/new-logic → develop

---

### 🔍 Summary
- 🧪 Coverage: 72.5%
- 🧱 Java Files Changed: 5
- 🗂️ Total Files Changed: 7
- ❌ Uncovered Lines: 14

---

### 📁 Notable Files Missing Coverage:
| File | Missing Lines |
|------|---------------|
src/main/java/com/example/Service.java | 8  
src/main/java/com/example/Helper.java | 6  

---

✅ Coverage analysis completed.
```

---

## 📦 Artifacts

* `diff.txt` – Git diff of changed files
* `jacoco.xml` – JaCoCo XML coverage report
* `coverage_report_raw.txt` – Raw `diff-cover` output
* `coverage_report.txt` – Clean markdown summary (posted on PR)
* `coverage_report.html` – Full HTML coverage report

---

## 🔒 Security Notes

* Git credentials are injected securely using Jenkins `withCredentials`.
* No sensitive information (like passwords/tokens) is committed into the repository.
* Always ensure your Bitbucket API tokens or credentials are rotated regularly.

---

## ✅ Benefits

* Enforces **quality in pull requests** by requiring tests for new/modified code.
* Prevents test coverage from regressing over time.
* Automates feedback loop with **instant PR comments** and **build statuses**.
* Provides developers with clear visibility into coverage gaps.

