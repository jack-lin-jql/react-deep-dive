# File structure

## Recommended way to structure React projects?

- React doesn't have an opinion on how one put files into folders
- There are a few common approaches popular in the ecosystem one may want to consider

### Grouping by features or routes

- Locate CSS, JS and tests together inside folders grouped by feature or route

```js
common/
  Avatar.js
  Avatar.css
  APIUtils.js
  APIUtils.test.js
feed/
  index.js
  Feed.js
  Feed.css
  FeedStory.js
  FeedStory.test.js
  FeedAPI.js
profile/
  index.js
  Profile.js
  ProfileHeader.js
  ProfileHeader.css
  ProfileAPI.js
```

### Grouping by file type

- Group similar files together

```js
api/
  APIUtils.js
  APIUtils.test.js
  ProfileAPI.js
  UserAPI.js
components/
  Avatar.js
  Avatar.css
  Feed.js
  Feed.css
  FeedStory.js
  FeedStory.test.js
  Profile.js
  ProfileHeader.js
  ProfileHeader.css
```

### Avoid too much nesting

- There're many pain points with deep directory nesting in JS projects
- Relative imports become harder to write
- Unless for a good reason, consider limiting to a max of 3 or 4 nested folders within a single project

### Don't overthink it

- Good idea to keep file that often change together close to each other, a principle called "colocation"