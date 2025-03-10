# Introduction

With Codeforces API you can get access to some of our data in machine-readable JSON format.

To access the data you just send a HTTP-request to address `https://codeforces.com/api/{methodName}` with method-specific parameters. Each method description has an example URL.

Each method call returns a JSON-object with three possible fields: status, comment and result.

- Status is either "OK" or "FAILED".
- If status is "FAILED" then comment contains the reason why the request failed. If status is "OK", then there is no comment.
- If status is "OK" then result contains method-dependent JSON-element which will be described for each method separately. If status is "FAILED", then there is no result.

API may be requested at most 1 time per two seconds. If you send more requests, you will receive a response with "FAILED" status and "Call limit exceeded" comment.

Language-depended fields like names or descriptions will be returned in the default language. Also, you can pass additional parameter `lang` with values `en` and `ru` to select the language of the result.

## Authorization

All methods can be requested anonymously. This way only public data will be accessable via API. To access data, private for some user (e.g. hacks during the contest), API key must be generated on https://codeforces.com/settings/api page. Each API key has two parameters: `key` and `secret`. To use the key you must add following parameters to your request.

1. `apiKey` — it must be equal to `key`
2. `time` — current time in unix format (e.g., System.currentTimeMillis()/1000). If the difference between server time and time, specified in parameter, will be greater than 5 minutes, request will be denied.
3. `apiSig` — signature to ensure that you know both `key` and `secret`. First six characters of the `apiSig` parameter can be arbitrary. We recommend to choose them at random for each request. Let's denote them as `rand`. The rest of the parameter is hexadecimal representation of SHA-512 hash-code of the following string: `<rand>/<methodName>?param1=value1&param2=value2...&paramN=valueN#<secret>` where `(param_1, value_1), (param_2, value_2),..., (param_n, value_n)` are all the request parameters (including `apiKey`, `time`, but excluding `apiSig`) with corresponding values, sorted lexicographically first by `param_i`, then by `value_i`.

For example:

If your `key` is `xxx`, `secret` is `yyy`, chosen `rand` is `123456` and you want to access method `contest.hacks` for contest 566, you should compose request like this: `https://codeforces.com/api/contest.hacks?contestId=566&apiKey=xxx&time=1741526578&apiSig=123456<hash>`, where `<hash>` is `sha512Hex(123456/contest.hacks?apiKey=xxx&contestId=566&time=1741526578#yyy)`

## JSONP

JSONP is supported. Add `jsonp` parameter to the query and the result will be returned as JavaScript function call.

For example, if `jsonp` parameter is `parseResponse` and method returned object `{"status":"OK","response":"..."}`, then final result is `parseResponse({"status":"OK","response":"..."});`.

# Methods

## <span id="blogEntry_comments">blogEntry.comments</span>

Returns a list of comments to the specified blog entry.
|Parameter|Description|
|:--|:--|
|**blogEntryId** (Required)|Id of the blog entry. It can be seen in blog entry URL. For example: [/blog/entry/79](https://codeforces.com/blog/entry/79)|

**Return value**: A list of [Comment](#Comment) objects.

**Example**: https://codeforces.com/api/blogEntry.comments?blogEntryId=79 

## <span id="blogEntry_view">blogEntry.view</span>

Returns blog entry.
|Parameter|Description|
|:--|:--|
|**blogEntryId** (Required)|Id of the blog entry. It can be seen in blog entry URL. For example: [/blog/entry/79](https://codeforces.com/blog/entry/79)|

**Return value**: Returns a [BlogEntry](#BlogEntry) object in full version.

**Example**: https://codeforces.com/api/blogEntry.view?blogEntryId=79 

## <span id="contest_hacks">contest.hacks</span>

Returns list of hacks in the specified contests. Full information about hacks is available only after some time after the contest end. During the contest user can see only own hacks.
|Parameter|Description|
|:--|:--|
|**contestId** (Required)|Id of the contest. It is not the round number. It can be seen in contest URL. For example: [/contest/566/status](https://codeforces.com/contest/566/status)|
|**asManager**|Boolean. If set to true, the response will contain information available to contest managers. Otherwise, the response will contain only the information available to the participants. You must be a contest manager to use it.|

**Return value**: Returns a list of [Hack](#Hack) objects.

**Example**: https://codeforces.com/api/contest.hacks?contestId=566&asManager=true 


## <span id="contest_list">contest.list</span>

Returns information about all available contests.
|Parameter|Description|
|:--|:--|
|**gym**|Boolean. If true — than gym contests are returned. Otherwide, regular contests are returned.|
|**groupCode**|Group code (e.g., `sfSJn5pz1a`) is used to filter contests. You need to log in with an account that has at least read access to the group.|

**Return value**: Returns a list of [Contest](#Contest) objects. If this method is called not anonymously, then all available contests for a calling user will be returned too, including mashups and private gyms.

**Example**: https://codeforces.com/api/contest.list?gym=true 

## <span id="contest_ratingChanges">contest.ratingChanges</span>

Returns rating changes after the contest.
|Parameter|Description|
|:--|:--|
|**contestId** (Required)|Id of the contest. It is **not** the round number. It can be seen in contest URL. For example: [/contest/566/status](https://codeforces.com/contest/566/status)|

**Return value**: Returns a list of [RatingChange](#RatingChange) objects.

**Example**: https://codeforces.com/api/contest.ratingChanges?contestId=566 

## <span id="contest_standings">contest.standings</span>

Returns the description of the contest and the requested part of the standings.
|Parameter|Description|
|:--|:--|
|**contestId** (Required)|Id of the contest. It is **not** the round number. It can be seen in contest URL. For example: [/contest/566/status](https://codeforces.com/contest/566/status)|
|**asManager**|Boolean. If set to true, the response will contain information available to contest managers. Otherwise, the response will contain only the information available to the participants. You must be a contest manager to use it.|
|**from**|1-based index of the standings row to start the ranklist.|
|**count**|Number of standing rows to return.|
|**handles**|Semicolon-separated list of handles. No more than 10000 handles is accepted.|
|**room**|If specified, than only participants from this room will be shown in the result. If not — all the participants will be shown.|
|**showUnofficial**|If true than all participants (virtual, out of competition) are shown. Otherwise, only official contestants are shown.|
|**participantTypes**|Comma-separated list of participant types without spaces. Possible values: CONTESTANT, PRACTICE, VIRTUAL, MANAGER, OUT_OF_COMPETITION. Only participants with the specified types will be displayed.|

**Return value**: Returns object with three fields: "contest", "problems" and "rows". Field "contest" contains a [RanklistRow](#RanklistRow) objects.

**Example**: https://codeforces.com/api/contest.standings?contestId=566&asManager=true&from=1&count=5&showUnofficial=true 

## <span id="contest_status">contest.status</span>

Returns submissions for specified contest. Optionally can return submissions of specified user.
|Parameter|Description|
|:--|:--|
|**contestId** (Required)|Id of the contest. It is **not** the round number. It can be seen in contest URL. For example: [/contest/566/status](https://codeforces.com/contest/566/status)|
|**asManager**|Boolean. If set to true, the response will contain information available to contest managers. Otherwise, the response will contain only the information available to the participants. You must be a contest manager to use it.|
|**handle**|Codeforces user handle.|
|**from**|1-based index of the first submission to return.|
|**count**|Number of returned submissions.|

**Return value**: Returns a list of [Submission](#Submission) objects, sorted in decreasing order of submission id.

**Example**: https://codeforces.com/api/contest.status?contestId=566&asManager=true&from=1&count=10 

## <span id="problemset_problems">problemset.problems</span>

Returns all problems from problemset. Problems can be filtered by tags.
|Parameter|Description|
|:--|:--|
|**tags**|Semicilon-separated list of tags.|
|**problemsetName**|Custom problemset's short name, like 'acmsguru'|

**Return value**: Returns two lists. List of [ProblemStatistics](#ProblemStatistics) objects.

**Example**: https://codeforces.com/api/problemset.problems?tags=implementation 

## <span id="problemset_recentStatus">problemset.recentStatus</span>

Returns recent submissions.
|Parameter|Description|
|:--|:--|
|**count** (Required)|Number of submissions to return. Can be up to 1000.|
|**problemsetName**|Custom problemset's short name, like 'acmsguru'|

**Return value**: Returns a list of [Submission](#Submission) objects, sorted in decreasing order of submission id.

**Example**: https://codeforces.com/api/problemset.recentStatus?count=10 

## <span id="recentActions">recentActions</span>

Returns recent actions.
|Parameter|Description|
|:--|:--|
|**maxCount** (Required)|Number of recent actions to return. Can be up to 100.|

**Return value**: Returns a list of [RecentAction](#RecentAction) objects.

**Example**: https://codeforces.com/api/recentActions?maxCount=30 

## <span id="user_blogEntries">user.blogEntries</span>

Returns a list of all user's blog entries.
|Parameter|Description|
|:--|:--|
|**handle** (Required)|Codeforces user handle.|

**Return value**: A list of [BlogEntry](#BlogEntry) objects in short form.

**Example**: https://codeforces.com/api/user.blogEntries?handle=Fefer_Ivan 

## <span id="user_friends">user.friends</span>

Returns authorized user's friends. Using this method requires authorization.
|Parameter|Description|
|:--|:--|
|**onlyOnline**|Boolean. If true — only online friends are returned. Otherwise, all friends are returned.|

**Return value**: Returns a list of strings — users' handles.

**Example**: https://codeforces.com/api/user.friends?onlyOnline=true 

## <span id="user_info">user.info</span>

Returns information about one or several users.
|Parameter|Description|
|:--|:--|
|**handles** (Required)|Semicolon-separated list of handles. No more than 10000 handles is accepted.|
|**checkHistoricHandles**|Boolean, the default value is true. If this flag is enabled, then use the history of handle changes when searching for a user.|

**Return value**: Returns a list of [User](#User) objects for requested handles.

**Example**: https://codeforces.com/api/user.info?handles=DmitriyH;Fefer_Ivan&checkHistoricHandles=false 

## <span id="user_ratedList">user.ratedList</span>

Returns the list users who have participated in at least one rated contest.
|Parameter|Description|
|:--|:--|
|**activeOnly**|Boolean. If true then only users, who participated in rated contest during the last month are returned. Otherwise, all users with at least one rated contest are returned.|
|**includeRetired**|Boolean. If true, the method returns all rated users, otherwise the method returns only users, that were online at last month.|
|**contestId**|Id of the contest. It is **not** the round number. It can be seen in contest URL. For example: [/contest/566/status](https://codeforces.com/contest/566/status)|

**Return value**: Returns a list of [User](#User) objects, sorted in decreasing order of rating.

**Example**: https://codeforces.com/api/user.ratedList?activeOnly=true&includeRetired=false 

## <span id="user_rating">user.rating</span>

Returns rating history of the specified user.
|Parameter|Description|
|:--|:--|
|**handle** (Required)|Codeforces user handle.|

**Return value**: Returns a list of [RatingChange](#RatingChange) objects for requested user.

**Example**: https://codeforces.com/api/user.rating?handle=Fefer_Ivan 

## <span id="user_status">user.status</span>

Returns submissions of specified user.
|Parameter|Description|
|:--|:--|
|**handle** (Required)|Codeforces user handle.|
|**from**|1-based index of the first submission to return.|
|**count**|Number of returned submissions.|

**Return value**: Returns a list of [Submission](#Submission) objects, sorted in decreasing order of submission id.

**Example**: https://codeforces.com/api/user.status?handle=Fefer_Ivan&from=1&count=10

# Return Object

## <span id="User">User</span>

Represents a Codeforces user.
|Field|Description|
|:--|:--|
|handle |String. Codeforces user handle.|
|email |String. Shown only if user allowed to share his contact info.|
|vkId |String. User id for VK social network. Shown only if user allowed to share his contact info.|
|openId |String. Shown only if user allowed to share his contact info.|
|firstName |String. Localized. Can be absent.|
|lastName |String. Localized. Can be absent.|
|country |String. Localized. Can be absent.|
|city |String. Localized. Can be absent.|
|organization |String. Localized. Can be absent.|
|contribution |Integer. User contribution.|
|rank |String. Localized.|
|rating |Integer.|
|maxRank |String. Localized.|
|maxRating |Integer.|
|lastOnlineTimeSeconds |Integer. Time, when user was last seen online, in unix format.|
|registrationTimeSeconds |Integer. Time, when user was registered, in unix format.|
|friendOfCount |Integer. Amount of users who have this user in friends.|
|avatar |String. User's avatar URL.|
|titlePhoto |String. User's title photo URL.|

## <span id="BlogEntry">BlogEntry</span>

Represents a Codeforces blog entry. May be in either short or full version.
|Field|Description|
|:--|:--|
|id |Integer.|
|originalLocale |String. Original locale of the blog entry.|
|creationTimeSeconds |Integer. Time, when blog entry was created, in unix format.|
|authorHandle |String. Author user handle.|
|title |String. Localized.|
|content |String. Localized. Not included in short version.|
|locale |String.|
|modificationTimeSeconds |Integer. Time, when blog entry has been updated, in unix format.|
|allowViewHistory |Boolean. If true, you can view any specific revision of the blog entry.|
|tags |String list.|
|rating |Integer.|

## <span id="Comment">Comment</span>

Represents a comment.
|Field|Description|
|:--|:--|
|id |Integer.|
|creationTimeSeconds |Integer. Time, when comment was created, in unix format.|
|commentatorHandle |String.|
|locale |String.|
|text |String.|
|parentCommentId |Integer. Can be absent.|
|rating |Integer.|

## <span id="RecentAction">RecentAction</span>

Represents a recent action.
|Field|Description|
|:--|:--|
|timeSeconds |Integer. Action time, in unix format.|
|blogEntry |[BlogEntry](#BlogEntry) object in short form. Can be absent.|
|comment |[Comment](#Comment) object. Can be absent.|

## <span id="RatingChange">RatingChange</span>

Represents a participation of user in rated contest.
|Field|Description|
|:--|:--|
|contestId |Integer.|
|contestName |String. Localized.|
|handle |String. Codeforces user handle.|
|rank |Integer. Place of the user in the contest. This field contains user rank on the moment of rating update. If afterwards rank changes (e.g. someone get disqualified), this field will not be update and will contain old rank.|
|ratingUpdateTimeSeconds |Integer. Time, when rating for the contest was update, in unix-format.|
|oldRating |Integer. User rating before the contest.|
|newRating |Integer. User rating after the contest.|

## <span id="Contest">Contest</span>

Represents a contest on Codeforces.
|Field|Description|
|:--|:--|
|id |Integer.|
|name |String. Localized.|
|type |Enum: CF, IOI, ICPC. Scoring system used for the contest.|
|phase |Enum: BEFORE, CODING, PENDING_SYSTEM_TEST, SYSTEM_TEST, FINISHED.|
|frozen |Boolean. If true, then the ranklist for the contest is frozen and shows only submissions, created before freeze.|
|durationSeconds |Integer. Duration of the contest in seconds.|
|freezeDurationSeconds |Integer. Can be absent. The ranklist freeze duration of the contest in seconds if any.|
|startTimeSeconds |Integer. Can be absent. Contest start time in unix format.|
|relativeTimeSeconds |Integer. Can be absent. Number of seconds, passed after the start of the contest. Can be negative.|
|preparedBy |String. Can be absent. Handle of the user, how created the contest.|
|websiteUrl |String. Can be absent. URL for contest-related website.|
|description |String. Localized. Can be absent.|
|difficulty |Integer. Can be absent. From 1 to 5. Larger number means more difficult problems.|
|kind |String. Localized. Can be absent. Human-readable type of the contest from the following categories: Official ICPC Contest, Official School Contest, Opencup Contest, School/University/City/Region Championship, Training Camp Contest, Official International Personal Contest, Training Contest.|
|icpcRegion |String. Localized. Can be absent. Name of the Region for official ICPC contests.|
|country |String. Localized. Can be absent.|
|city |String. Localized. Can be absent.|
|season |String. Can be absent.|

## <span id="Party">Party</span>

Represents a party, participating in a contest.
|Field|Description|
|:--|:--|
|contestId |Integer. Can be absent. Id of the contest, in which party is participating.|
|members |List of [Member](#Member) objects. Members of the party.|
|participantType |Enum: CONTESTANT, PRACTICE, VIRTUAL, MANAGER, OUT_OF_COMPETITION.|
|teamId |Integer. Can be absent. If party is a team, then it is a unique team id. Otherwise, this field is absent.|
|teamName |String. Localized. Can be absent. If party is a team or ghost, then it is a localized name of the team. Otherwise, it is absent.|
|ghost |Boolean. If true then this party is a ghost. It participated in the contest, but not on Codeforces. For example, Andrew Stankevich Contests in Gym has ghosts of the participants from Petrozavodsk Training Camp.|
|room |Integer. Can be absent. Room of the party. If absent, then the party has no room.|
|startTimeSeconds |Integer. Can be absent. Time, when this party started a contest.|

## <span id="Member">Member</span>

Represents a member of a party.
|Field|Description|
|:--|:--|
|handle |String. Codeforces user handle.|
|name |String. Can be absent. User's name if available.|

## <span id="Problem">Problem</span>

Represents a problem.
|Field|Description|
|:--|:--|
|contestId |Integer. Can be absent. Id of the contest, containing the problem.|
|problemsetName |String. Can be absent. Short name of the problemset the problem belongs to.|
|index |String. Usually, a letter or letter with digit(s) indicating the problem index in a contest.|
|name |String. Localized.|
|type |Enum: PROGRAMMING, QUESTION.|
|points |Floating point number. Can be absent. Maximum amount of points for the problem.|
|rating |Integer. Can be absent. Problem rating (difficulty).|
|tags |String list. Problem tags.|

## <span id="ProblemStatistics">ProblemStatistics</span>

Represents a statistic data about a problem.
|Field|Description|
|:--|:--|
|contestId |Integer. Can be absent. Id of the contest, containing the problem.|
|index |String. Usually, a letter or letter with digit(s) indicating the problem index in a contest.|
|solvedCount |Integer. Number of users, who solved the problem.|

## <span id="Submission">Submission</span>

Represents a submission.
|Field|Description|
|:--|:--|
|id |Integer.|
|contestId |Integer. Can be absent.|
|creationTimeSeconds |Integer. Time, when submission was created, in unix-format.|
|relativeTimeSeconds |Integer. Number of seconds, passed after the start of the contest (or a virtual start for virtual parties), before the submission.|
|problem |[Problem](#Problem) object.|
|author |[Party](#Party) object.|
|programmingLanguage |String.|
|verdict |Enum: FAILED, OK, PARTIAL, COMPILATION_ERROR, RUNTIME_ERROR, WRONG_ANSWER, WRONG_ANSWER, TIME_LIMIT_EXCEEDED, MEMORY_LIMIT_EXCEEDED, IDLENESS_LIMIT_EXCEEDED, SECURITY_VIOLATED, CRASHED, INPUT_PREPARATION_CRASHED, CHALLENGED, SKIPPED, TESTING, REJECTED. Can be absent.|
|testset |Enum: SAMPLES, PRETESTS, TESTS, CHALLENGES, TESTS1, ..., TESTS10. Testset used for judging the submission.|
|passedTestCount |Integer. Number of passed tests.|
|timeConsumedMillis |Integer. Maximum time in milliseconds, consumed by solution for one test.|
|memoryConsumedBytes |Integer. Maximum memory in bytes, consumed by solution for one test.|
|points |Floating point number. Can be absent. Number of scored points for IOI-like contests.|

## <span id="Hack">Hack</span>

Represents a hack, made during Codeforces Round.
|Field|Description|
|:--|:--|
|id |Integer.|
|creationTimeSeconds |Integer. Hack creation time in unix format.|
|hacker |[Party](#Party) object.|
|defender |[Party](#Party) object.|
|verdict |Enum: HACK_SUCCESSFUL, HACK_UNSUCCESSFUL, INVALID_INPUT, GENERATOR_INCOMPILABLE, GENERATOR_CRASHED, IGNORED, TESTING, OTHER. Can be absent.|
|problem |[Problem](#Problem) object. Hacked problem.|
|test |String. Can be absent.|
|judgeProtocol |Object with three fields: "manual", "protocol" and "verdict". Field manual can have values "true" and "false". If manual is "true" then test for the hack was entered manually. Fields "protocol" and "verdict" contain human-readable description of judge protocol and hack verdict. Localized. Can be absent.|

## <span id="RanklistRow">RanklistRow</span>

Represents a ranklist row.
|Field|Description|
|:--|:--|
|party |[Party](#Party) object. Party that took a corresponding place in the contest.|
|rank |Integer. Party place in the contest.|
|points |Floating point number. Total amount of points, scored by the party.|
|penalty |Integer. Total penalty (in ICPC meaning) of the party.|
|successfulHackCount |Integer.|
|unsuccessfulHackCount |Integer.|
|problemResults |List of [ProblemResult](#ProblemResult) objects. Party results for each problem. Order of the problems is the same as in "problems" field of the returned object.|
|lastSubmissionTimeSeconds |Integer. For IOI contests only. Time in seconds from the start of the contest to the last submission that added some points to the total score of the party. Can be absent.|

## <span id="ProblemResult">ProblemResult</span>

Represents a submissions results of a party for a problem.
|Field|Description|
|:--|:--|
|points |Floating point number.|
|penalty |Integer. Penalty (in ICPC meaning) of the party for this problem. Can be absent.|
|rejectedAttemptCount |Integer. Number of incorrect submissions.|
|type |Enum: PRELIMINARY, FINAL. If type is PRELIMINARY then points can decrease (if, for example, solution will fail during system test). Otherwise, party can only increase points for this problem by submitting better solutions.|
|bestSubmissionTimeSeconds |Integer. Number of seconds after the start of the contest before the submission, that brought maximal amount of points for this problem. Can be absent.|

---

BY: WHANIA
UPD: 2025/03/09
REF: https://codeforces.com/apiHelp
