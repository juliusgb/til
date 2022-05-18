# How to access GitHub Maven packages via GitHub actions

Steps:

- Add a Personal Access Token (PAT) to your GitHub Repository with permissions `repo, workflow, read:packages`
- Add that token as repository secret named `GHA_MVN_PKGS_READ`: Repository -> Settings -> Secrets
- Add repository settings to project's `pom.xml`. For GitHub Enterprise (<https://docs.github.com/en/enterprise-server@3.5/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry>), set the url to `https://maven.HOSTNAME/OWNER/REPO` or even `https://maven.HOSTNAME/OWNER/*`.

```xml
...
  <repository>
    <!-- IMPORTANT: id must be the same as in the workflow yml -->
    <id>github-maven-packages</id>
    <name>GitHub Maven Packages</name>
    <!-- NOTE: GH-Enterprise uses prefix https://maven.HOSTNAME/
        e.g., https://maven.github.myawesomeco.com/ 
     -->
    <url>https://maven.pkg.github.com/<my-username>/</url> 
  </repository>
```

- Setup the `actions/setup-java@v2` - like the docs (<https://github.com/actions/setup-java/blob/main/docs/advanced-usage.md#Publishing-using-Apache-Maven>) say:

```yml
. . .
  - name: Set up JDK 11
    uses: actions/setup-java@v2
    with:
      java-version: '11'
      distribution: 'temurin'
      server-id: github-maven-package

  - name: Print settings xml
    run: |
      $LocalMavenM2Dir = "$env:USERPROFILE\.m2\"
      type $LocalMavenM2Dir\settings.xml

  - name: build and unit test
    run: mvn -B clean test    # pulls in the dependencies
    env:
      GITHUB_ACTOR: juliusgb  # can be github org or github username
      # prefer your own instead of secrets.GITHUB_TOKEN (easier to track and can access private repos)
      GITHUB_TOKEN: ${{ secrets.GHA_MVN_PKGS_READ }}
```

GitHub uses the env variables `GITHUB_ACTOR` and `GITHUB_TOKEN` as the default credentials when it generates the `settings.xml`.

To see what the generated `settings.xml` looks like is what the step `Print settings xml` does (on Windows).
If you're on Linux, adjust the 2 lines after `run: |`.
