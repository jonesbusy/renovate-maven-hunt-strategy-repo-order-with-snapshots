# maven-hunt-strategy-repo-order-with-snapshots

In order to reproduce and test maven hunt strategy with snapshot repositories before release repositories

At the time of writting the dependency should be bumped from `5.5.2` to `8.14.1` at least.

This can be confirmed using versions-maven-plugin

```
mvn -X versions:display-dependency-updates "-Dmaven.version.ignore=.*-M.*,.*-m.*,.*-alpha.*,.*-rc.*"
```

That will output

```
[INFO]   com.atlassian.bitbucket.server:bitbucket-access-tokens ...
[INFO]                                                          5.5.2 -> 8.14.1
```

# Renovate

Test locally

````
npm install -g renovate
export RENOVATE_ONBOARDING=false
export LOG_LEVEL=debug
export RENOVATE_CONFIG_FILE=renovate.json
export RENOVATE_CACHE_HARD_TTL_MINUTES=0
````

````
renovate --platform=local
````

## Output (partial)

This is expected because not available in maven central

```
DEBUG: Content is not found for Maven url (repository=local)
       "url": "https://repo.maven.apache.org/maven2/com/atlassian/bitbucket/server/bitbucket-access-tokens/maven-metadata.xml",
       "statusCode": undefined
```

This is NOT expected because we expect to lookup on release repository

```
DEBUG: Found 344 new releases for com.atlassian.bitbucket.server:bitbucket-access-tokens in repository https://packages.atlassian.com/mvn/maven-atlassian-snapshot-external/ (repository=local)
```

And of course renovate doesn't find any updates

```
{
    "datasource": "maven",
    "depName": "com.atlassian.bitbucket.server:bitbucket-access-tokens",
    "currentValue": "5.5.2",
    "fileReplacePosition": 696,
    "registryUrls": [
        "https://repo.maven.apache.org/maven2",
        "https://packages.atlassian.com/mvn/maven-atlassian-snapshot-external",
        "https://packages.atlassian.com/mvn/maven-atlassian-external"
    ],
    "depType": "compile",
    "updates": [],
    "packageName": "com.atlassian.bitbucket.server:bitbucket-access-tokens",
    "versioning": "maven",
    "warnings": [],
    "sourceUrl": "http://stash.dev.internal.atlassian.com/git/STASH/stash.git",
    "registryUrl": "https://packages.atlassian.com/mvn/maven-atlassian-snapshot-external",
    "currentVersion": "5.5.2",
    "fixedVersion": "5.5.2"
}
```

In this case we should expect `https://packages.atlassian.com/mvn/maven-atlassian-snapshot-external` to be filtered on `registryUrls` because dependency to renovate is not a SNAPSHOT

If we removate or change the order of the repository we can find upgrades

```
DEBUG: Looking up com.atlassian.bitbucket.server:bitbucket-access-tokens in repository https://packages.atlassian.com/mvn/maven-atlassian-external/ (repository=local)
```

```
"updates": [
    {
        "bucket": "non-major",
        "newVersion": "5.16.11",
        "newValue": "5.16.11",
        "releaseTimestamp": "2019-12-18T06:06:11.000Z",
        "newMajor": 5,
        "newMinor": 16,
        "updateType": "minor",
        "branchName": "renovate/com.atlassian.bitbucket.server-bitbucket-access-tokens-5.x"
    },
    {
        "bucket": "major",
        "newVersion": "8.14.1",
        "newValue": "8.14.1",
        "releaseTimestamp": "2023-10-09T05:01:44.000Z",
        "newMajor": 8,
        "newMinor": 14,
        "updateType": "major",
        "branchName": "renovate/com.atlassian.bitbucket.server-bitbucket-access-tokens-8.x"
    }
],
```
