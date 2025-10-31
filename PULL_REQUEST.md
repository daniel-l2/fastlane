## [spaceship][deliver][pilot] Add support for App Clips default experience metadata and review information with default folder

### Checklist

- [x] I've run `bundle exec rspec` from the root directory to see all new and existing tests pass
- [x] I've followed the _fastlane_ code style and run `bundle exec rubocop -a` to ensure the code style is valid
- [ ] I see several green `ci/circleci` builds in the "All checks have passed" section of my PR
- [x] I've read the [Contribution Guidelines](https://github.com/fastlane/fastlane/blob/master/CONTRIBUTING.md)
- [x] I've updated the documentation if necessary.
- [x] I've added or updated relevant unit tests.

### Motivation and Context

This PR re-opens #22041 with additional improvements. It includes **all the original App Clip support features** from #22041 plus new `default` folder support for header images and metadata.

**Re-opening the App Clip PR:**

The original PR #22041 has been stalled without merge since 2024. Our team has been using these App Clip features in production and they work reliably. Since the PR remains unmerged and our team (and potentially others) need these features, I'm re-opening this work with additional improvements.

**What's included from #22041:**
- App Clip default experience metadata support (subtitle, action)
- Localized App Clip header images (1800x1200)
- App Clip review information
- Pilot beta App Clip invocation support
- Spaceship API updates for App Clip resources
- Automatic metadata copying from live versions

**New addition - Default folder support:**

Building upon #22041, this PR adds support for a `default` folder in App Clip header images and metadata, bringing App Clip localization in line with fastlane's existing patterns for screenshots and regular app metadata.

Users can now:
1. Place content in a `default` folder
2. Have it automatically assigned to all enabled languages without specific localized content
3. Override default content with language-specific versions when needed

### Description

**All changes from #22041 (App Clip core functionality):**

1. **Deliver - App Clip metadata support**
   - Specify App Clip action (PLAY, VIEW, OPEN, etc.)
   - Localized subtitle support
   - Localized header images (1800x1200 requirement enforced)
   - App Clip review information
   - Automatic metadata copying from live versions when no options specified
   - Download command for App Clip header images

2. **Pilot - Beta testing support**
   - Beta App Clip invocation specification

3. **Spaceship - API layer updates**
   - New API endpoints for App Clip resources
   - Updated request handling for App Clip default experience
   - App Clip localization management

**New changes in this PR (Default folder support):**

4. **Default folder support for App Clip header images** (`deliver/lib/deliver/loader.rb`)
   - `LanguageFolder` now correctly handles the `default` folder (language = nil)
   - Header image validation filters out nil languages to allow multiple images in default folder
   - Automatic assignment of default images to languages without specific images

5. **Default folder logic for App Clip metadata** (`deliver/lib/deliver/upload_app_clip_default_experience_header_images.rb`)
   - New `assign_default_images` method that detects enabled languages and assigns default header image
   - Identifies default folder by path inspection (not relying on nil language)
   - Removes default images from upload list after assignment

**How it works:**

```
app-clip-header-images/
  ├── default/          # This image will be used for all languages
  │   └── header.jpg
  ├── en-US/           # Optional: override for specific language
  │   └── header.jpg
  └── ko/              # This will use default/header.jpg
```

The same pattern applies to metadata folders.

### Testing Steps

1. Create an app with App Clip target
2. Set up folder structure:
   ```
   fastlane/app-clip-header-images/
     default/
       header.jpg (1800x1200)
   ```
3. Configure `Deliverfile`:
   ```ruby
   languages(['en-US', 'ko', 'ja'])
   app_clip_default_experience_subtitle({
     "default" => "Quick access to our app"
   })
   app_clip_default_experience_action("VIEW")
   ```
4. Run `fastlane deliver`
5. Verify in App Store Connect that all three languages have the header image and subtitle

**Expected behavior:**
- Default header image appears for all enabled languages
- Languages with specific images use those instead of default
- Metadata follows the same pattern as the original #22041 implementation
- No duplicate validation errors for default folder

---

**Note:** This PR is based on and extends #22041. Credit for the core App Clip functionality goes to the original contributors. I'm reopening this work with added default folder support since the original PR hasn't been merged and our team relies on these features in production.

Related: #17639, #22041
