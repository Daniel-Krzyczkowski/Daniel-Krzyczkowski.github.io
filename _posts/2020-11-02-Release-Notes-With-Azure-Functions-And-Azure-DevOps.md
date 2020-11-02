---
title: "Release notes with Azure Functions and Azure DevOps"
excerpt: "This article presents how to generate release notes with Azure Functions in Azure DevOps"
header:
  image: /images/devisland/article46/assets/ReleaseManagementWiki1.png
---

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki1.png?raw=true" alt="Communication between microservices with Azure Service Bus"/>
</p>

# Introduction

It does not matter what kind of project you run, there is always nice to have release notes. Collection of implemented user stories, features, and solved bugs. Of course, after each release, we can write these release notes manually but there is a better way to automate this process. In one of the projects together with the team, we use Azure DevOps together with Wiki as a source of information about the project. Why not use the project's wiki to store release notes? In this article, I would like to present how to automate generating release notes for the project using Azure Functions and Azure DevOps.

The end result can be like presented below:

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki12.PNG?raw=true" alt="Image not found"/>
</p>


# Wiki in the Azure DevOps

Let's start with the Azure DevOps Wiki for the project. As you can see below, I published a Wiki as GIT repository:

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki2.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki3.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki4.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki5.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki6.PNG?raw=true" alt="Image not found"/>
</p>


# Branching strategy and merges

In this specific scenario I used GIT flow branching strategy. Below diagram presents how releases to specific environments are mapped on branches from the GIT repository:

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki18.png?raw=true" alt="Image not found"/>
</p>

There are three environments used:

1. DEV
2. TEST
3. PRODUCTION

At some point, there is *release* branch created with the name: *release-1.0.0*. After tests on the TEST environment, and solving the bugs, there is a pull request created to merge changes to the master branch and release packages on the PROD environment. After this merge, I want to have release notes generated in the Wiki of the Azure DevOps project.

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki11.PNG?raw=true" alt="Image not found"/>
</p>


# Azure Function App responsibility

Azure Functions are perfect for the scenarios where we want to handle events - in this specific case we want to trigger Azure Function, once there is a successful merge to the *master* branch. This Azure Function uses Azure DevOps API to:

1. Pull all User Stories and Bugs that were completed in the release (in this example *release-1.0.0*)
2. Create release notes structure
3. Publish release notes to the Azure DevOps project's wiki

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki7.PNG?raw=true" alt="Image not found"/>
</p>

Let's talk about implementation details. First of all, to communicate with the Azure DevOps API we need to obtaind [Personal Access Token (PAT)](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page):

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki8.PNG?raw=true" alt="Image not found"/>
</p>

When creating new Personal Access Token, we have to indicate scopes so we can get information about work items from the backlog and gain access to modify project's wiki pages:

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki9.PNG?raw=true" alt="Image not found"/>
</p>

In the source code of the Function App, I use *HttpClient* to make calls to the Azure DevOps API. Authentication is done using PAT token in the *Authorization* header:

```csharp
        private void AuthenticateRequest()
        {
            _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic",
                                               Convert.ToBase64String(ASCIIEncoding.ASCII.GetBytes(
                                               string.Format("{0}:{1}", "", _azureDevOpsConfiguration.PersonalAccessToken))));
        }
```

Then I have created two methods, one to get user stories attached to the release and one to get resolved bugs:

```csharp
        private async Task<UserStoryDetails> GetUserStoryDetailsAsync(JObject pullRequestWorkItemDetails, string workItemUrl)
        {
            var fields = pullRequestWorkItemDetails["fields"];
            var relations = pullRequestWorkItemDetails["relations"];
            var userStoryId = pullRequestWorkItemDetails["id"].ToString();
            var userStoryTitle = fields["System.Title"].ToString();
            var userStoryDescription = fields["System.Description"].ToString();
            var userStoryAcceptanceCriteria = fields["Microsoft.VSTS.Common.AcceptanceCriteria"].ToString();

            var userStoryDetails = new UserStoryDetails()
            {
                Id = userStoryId,
                Title = userStoryTitle,
                Description = userStoryDescription,
                AcceptanceCriteria = userStoryAcceptanceCriteria,
                Url = workItemUrl
            };

            foreach (var relation in relations.Children())
            {
                if (relation["rel"].ToString() == "System.LinkTypes.Hierarchy-Reverse")
                {
                    var parentUrl = relation["url"].ToString();
                    var parentDetailsResponse = await _httpClient.GetAsync(parentUrl);
                    var parentDetailsResponseAsString = await parentDetailsResponse.Content.ReadAsStringAsync();

                    JObject parentWorkItemDetails = JsonConvert.DeserializeObject<JObject>(parentDetailsResponseAsString);
                    var parentFields = parentWorkItemDetails["fields"];
                    var parentTitle = parentFields["System.Title"].ToString();
                    userStoryDetails.ParentFeatureTitle = parentTitle;
                }
            }

            return userStoryDetails;
        }
```

```csharp
        private async Task<BugDetails> GetBugDetailsAsync(JObject pullRequestWorkItemDetails, string workItemUrl)
        {
            var fields = pullRequestWorkItemDetails["fields"];
            var bugId = pullRequestWorkItemDetails["id"].ToString();
            var bugTtitle = fields["System.Title"].ToString();
            var reproductionSteps = fields["Microsoft.VSTS.TCM.ReproSteps"].ToString();

            var bugDetails = new BugDetails()
            {
                Id = bugId,
                Title = bugTtitle,
                ReproductionSteps = reproductionSteps,
                Url = workItemUrl
            };

            var bugRelations = pullRequestWorkItemDetails["relations"];
            foreach (var relation in bugRelations.Children())
            {
                if (relation["rel"].ToString() == "System.LinkTypes.Hierarchy-Reverse")
                {
                    var parentUrl = relation["url"].ToString();
                    var parentDetailsResponse = await _httpClient.GetAsync(parentUrl);
                    var parentDetailsResponseAsString = await parentDetailsResponse.Content.ReadAsStringAsync();

                    JObject parentWorkItemDetails = JsonConvert.DeserializeObject<JObject>(parentDetailsResponseAsString);
                    var parentFields = parentWorkItemDetails["fields"];
                    var parentTitle = parentFields["System.Title"].ToString();
                    bugDetails.ParentUserStoryTitle = parentTitle;
                }
            }

            return bugDetails;
        }
```

Now to generate the release notes in the Wiki, I wrote below method:

```csharp
        private async Task UpdateWikiReleaseNotesAsync(ReleaseNotesReport releaseNotesReport)
        {
            StringBuilder sb = new StringBuilder();
            sb.Append($"# {releaseNotesReport.ReleaseTitle}");
            sb.Append("\n");
            sb.Append($"## Release notes for the release: {releaseNotesReport.ReleaseVersion}\n");
            sb.Append("\n");
            sb.Append("## Repository name:\n");
            sb.Append(releaseNotesReport.RepositoryName);
            sb.Append("\n");

            sb.Append("## User Stories completed:\n");
            sb.Append("\n");
            foreach (var userStory in releaseNotesReport.ReleaseWorkItems.UserStories)
            {
                sb.Append($"[{userStory.Title}](https://dev.azure.com/{_azureDevOpsConfiguration.Organization}/{_azureDevOpsConfiguration.Project}/_workitems/edit/{userStory.Id})");
                sb.Append("\n");
                sb.Append("\n");
            }

            sb.Append("\n");

            sb.Append("## Bugs fixed:\n");
            sb.Append("\n");
            foreach (var bug in releaseNotesReport.ReleaseWorkItems.Bugs)
            {
                sb.Append($"[{bug.Title}](https://dev.azure.com/{_azureDevOpsConfiguration.Organization}/{_azureDevOpsConfiguration.Project}/_workitems/edit/{bug.Id})");
                sb.Append("\n");
                sb.Append("\n");
            }

            var wikiReleaseNotes = new WikiReleaseNotes()
            {
                Content = sb.ToString()
            };

            await _httpClient.PutAsJson($"https://dev.azure.com/{_azureDevOpsConfiguration.Organization}/{_azureDevOpsConfiguration.Project}/" +
                                           $"_apis/wiki/wikis/{_azureDevOpsConfiguration.WikiId}/" +
                                           $"pages?path=Releases/{releaseNotesReport.ReleaseVersion}" +
                                           $"&versionDescriptor.version=master" +
                                           $"&api-version={_azureDevOpsConfiguration.ApiVersion}", wikiReleaseNotes);
        }
```

How the Function App is triggered? I used *HttpTrigger* function that receives payload from the Azure DevOps when webhook is called after the merge. Two important parameters are:

1. Source code repository ID
2. Pull request ID


```csharp
        [FunctionName(FunctionsRegistry.UipReleaseNotesFuncAppName)]
        public async Task<IActionResult> RunAsync(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            var pullRequestMergeEvent = JsonConvert.DeserializeObject<PullRequestMergeEvent>(requestBody);

            if (pullRequestMergeEvent != null)
            {
                await _azureDevOpsService.GetPullRequestContentAsync(pullRequestMergeEvent.resource.repository.id,
                                                                     pullRequestMergeEvent.resource.pullRequestId);
                return new OkResult();
            }

            else
            {
                return new BadRequestResult();
            }
        }
```


# Azure DevOps webhook

Once Azure Function App is published on the Azure cloud, we can setup web hook in the Azure DevOps to call it once there is successful merge from the *release-1.0.0* branch to the *master* branch:

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki13.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki14.PNG?raw=true" alt="Image not found"/>
</p>

We can apply filters, in this case I set filter on the repository, branch and merge result to make sure that Azure Function is called only after successful merge to the *master* branch:

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki15.PNG?raw=true" alt="Image not found"/>
</p>

In the last step we have to provide the URL address of the Azure Function App (together with the code):

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki16.PNG?raw=true" alt="Image not found"/>
</p>


# Final result

Below I presented the final result. Once pull request is completed, there is a merge to *master* branch and Function App is then triggered. All user stories and bugs are then listed in the Wiki:

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki17.PNG?raw=true" alt="Image not found"/>
</p>

<p align="center">
<img src="/images/devisland/article46/assets/ReleaseManagementWiki12.PNG?raw=true" alt="Image not found"/>
</p>

# Summary

In this article, I presented the approach to automate generating release notes in the project's wiki in the Azure DevOps using Azure Function Apps and webhooks. Of course, the whole process can be adjusted to many different scenarios - the main idea here is to show that without any special extensions we can achieve release notes.
