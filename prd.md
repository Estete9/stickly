# Product Requirements Document (PRD): Sticker Pack Creator for WhatsApp

1.  **Product Overview**
    1.1 **Product Name**
    Stickly WhatsApp sticker Organizer

    1.2 **Objective**
    Enable users to organize their existing WhatsApp stickers into custom packs, enhancing communication efficiency by making stickers easily accessible and searchable within WhatsApp conversations. The app dynamically generates contents.json metadata for Android with robust validation, prioritizing emoji tagging for searchability and accessibility text for inclusivity.

    1.3 **Target Platforms**
    Initial Release: Android
    Future Expansion: iOS

    1.4 **Target Audience**
    WhatsApp users with large sticker collections (thousands to tens of thousands), seeking faster access to stickers during conversations.

2.  **Background and Problem Statement**
    2.1 **Problem**
    WhatsApp users with extensive sticker libraries struggle to find specific stickers quickly during chats, slowing communication. Existing stickers in WhatsApp’s restricted directory are not easily reorganized or searchable beyond basic pack navigation.

    2.2 **Solution**
    A Flutter-based app that:
    - Accesses WhatsApp’s sticker directory via Storage Access Framework (SAF).
    - Allows users to create custom sticker packs with metadata dynamically generated as `contents.json`.
    - Ensures searchability through comprehensive emoji tagging (1-3 per sticker).
    - Integrates packs into WhatsApp with validation for compliance.
    - Provides in-app filtering to streamline sticker selection.

    2.3 **Key Differentiators**
    - Reuses existing WhatsApp stickers with dynamic metadata generation.
    - Prioritizes emoji-based searchability and accessibility text for inclusivity.
    - Handles large sticker volumes with performance optimizations.

3.  **Features and Requirements**
    3.1 **Core Features**
        3.1.1 **Sticker Access**
        - Description: Access WhatsApp’s sticker directory to display all `.webp` files.
        - Requirements:
            - Use SAF (`storage_access_framework`) to prompt users for `/Android/media/com.whatsapp/WhatsApp/Media/WhatsApp Stickers`.
            - Cache directory URI in SharedPreferences. Handle URI becoming invalid.
            - Display stickers in a lazy-loaded GridView (e.g., `ListView.builder`, 50 stickers per page).
        - Success Criteria: Users can view all WhatsApp stickers without errors across Android 10-14. Cached URI works reliably.

        3.1.2 **In-App Filtering**
        - Description: Filter stickers by pack name, tags, and date to aid selection.
        - Requirements:
            - Search bar (`TextField`) with filter options:
                - Pack Name/Category: Matches user-defined pack names.
                - Tags: User-assigned tags stored in `sqflite`.
                - Date Downloaded: Approximated via SAF’s `DocumentFile.lastModified()`. Handle cases where date is unavailable.
            - UI: Dropdown or filter chips for selecting filter type, with real-time grid updates.
        - Success Criteria: Filters reduce displayed stickers accurately; performance remains smooth with 10,000+ stickers. Filtering handles missing dates gracefully.

        3.1.3 **Pack Creation**
        - Description: Create custom sticker packs with validated metadata for searchability and inclusivity.
        - Requirements:
            - Multi-select via long-press or checkboxes. Handle rapid selection/deselection.
            - Dialog for:
                - Pack Name: Mandatory, max 128 chars, user-entered. Validate input (e.g., prevent empty, handle excessive whitespace).
                - Identifier: Auto-generated (e.g., UUID or userID+timestamp), max 128 chars, alphanumeric + "_", "-", ".", " ". Ensure uniqueness.
                - Publisher: User-entered or default to app name, max 128 chars. Validate input.
                - Tags: Optional, stored in `sqflite` for app filtering.
                - Emojis per Sticker: Mandatory, 1-3 emojis via emoji picker, emphasized for searchability. Enforce selection. Handle newer/uncommon emojis.
                - Accessibility Text: Optional (encouraged), max 125 chars, text input per sticker for inclusivity.
            - Tray image: First selected sticker, resized to 96x96 pixels as PNG, max 50KB (`image` package). Handle failure if first sticker is invalid/corrupt.
            - Copy `.webp` files to `sticker_packs/<identifier>/` in app’s documents directory. Handle potential file I/O errors (permissions, out of space).
            - Generate `contents.json` with validation (see 3.3.1).
            - Validate stickers: 512x512 pixels, max 100KB (static). Clearly inform user about invalid stickers excluded from pack. Handle exact boundary conditions (e.g., exactly 100KB).
            - Add pack to WhatsApp via `flutter_whatsapp_stickers`. Handle cases where WhatsApp is not installed or integration fails.
            - Enforce pack limits (min 1 sticker, max 30 stickers).
        - Success Criteria: Packs appear in WhatsApp with correct metadata; 100% of stickers have 1-3 emojis; creation completes in <5 seconds for 30 stickers. Errors are handled gracefully with user feedback.

        3.1.4 **WhatsApp Searchability**
        - Description: Ensure stickers are searchable in WhatsApp via emojis.
        - Requirements:
            - Prompt users to assign 1-3 emojis per sticker (mandatory, with UI emphasis).
            - Include emojis in `contents.json` under `stickers` array.
            - Use descriptive pack names and test file names for word-based search.
            - Order stickers in `contents.json` to match selection order.
        - Success Criteria: Users can find 100% of emoji-assigned stickers in WhatsApp; word search works if supported.

        3.1.5 **Performance Optimization**
        - Description: Handle large sticker collections efficiently.
        - Requirements:
            - Lazy load stickers in grid (50 per page).
            - Cache filter results in memory or `sqflite`.
            - Limit pack size to 30 stickers.
            - Optimize image loading/display in grid.
        - Success Criteria: App remains responsive with 10,000+ stickers; grid loads in <2 seconds per page. Scrolling is smooth.

    3.2 **User Interface**
        3.2.1 **Home Screen**
        - Lazy-loaded GridView of stickers.
        - Search bar with filter dropdown/chips.
        - Clear indication of loading state or errors.

        3.2.2 **Pack Creation Dialog**
        - Fields: Pack name, identifier (auto-filled), publisher, tags, emoji picker (1-3 per sticker, mandatory with tooltip), accessibility text (optional with encouragement text).
        - Bulk emoji option: Apply same emojis to all selected stickers.
        - Clear feedback on validation errors (e.g., missing emojis, name too long).

        3.2.3 **Onboarding**
        - Guided flow: Text + screenshot (“Open SAF → Navigate to WhatsApp → Select Stickers folder”).
        - Handle case where user selects the wrong folder.

    3.3 **Technical Requirements**
        3.3.1 **Metadata Structure (`contents.json`)**
        - Generation: Dynamically created per pack creation with validation.
        - Location: `sticker_packs/<identifier>/contents.json` in app’s documents directory.
        - Structure:
            ```json
            {
              "sticker_packs": [
                {
                  "identifier": "user123_1698765432",
                  "name": "Funny Pack",
                  "publisher": "Sticker Pack Creator",
                  "tray_image_file": "tray_icon.png",
                  "image_data_version": "1.0",
                  "avoid_cache": false,
                  "stickers": [
                    {
                      "image_file": "sticker1.webp",
                      "emojis": ["😂", "🤓"],
                      "accessibility_text": "Laughing face"
                    },
                    {
                      "image_file": "sticker2.webp",
                      "emojis": ["😢"],
                      "accessibility_text": "Crying face"
                    }
                  ]
                }
              ]
            }
            ```
        - Mandatory Fields:
            - `identifier`
            - `name`
            - `publisher`
            - `tray_image_file`
            - `avoid_cache`
            - `stickers` (with `image_file`, `emojis` [1-3])
        - Optional Fields:
            - `image_data_version` (default "1.0")
            - `accessibility_text` (encouraged)
        - Validation:
            - Ensure JSON format, field lengths, and emoji count (1-3) before saving and attempting WhatsApp integration.

        3.3.2 **Asset Requirements**
        - Stickers:
            - Format: `.webp`
            - Dimensions: 512x512 pixels
            - Max Size: 100KB (static)
        - Tray Image:
            - Format: `.png`
            - Dimensions: 96x96 pixels
            - Max Size: 50KB
        - Packages:
            - `storage_access_framework`: SAF integration
            - `sqflite`: Local database for tags/metadata
            - `flutter_whatsapp_stickers`: Android WhatsApp integration
            - `image`: Tray image resizing/conversion
            - `shared_preferences`: URI caching
            - `emoji_picker_flutter`: Emoji selection
            - `uuid`: Unique identifier generation
            - `path_provider`: Accessing app directories

4.  **User Flow**
    1.  Launch: App checks for cached URI. If missing/invalid, prompts SAF selection → User picks WhatsApp Stickers folder.
    2.  Browse: View stickers in grid → Optionally Filter by name/tags/date.
    3.  Create: Multi-select stickers (1-30) → Trigger pack creation → Enter pack details → Assign 1-3 emojis per sticker → Confirm.
    4.  Integrate: App validates stickers & metadata → Generates validated `contents.json` → Copies files → Calls WhatsApp integration.
    5.  Use: Open WhatsApp → Search stickers by emojis/pack name.

5.  **Platform Considerations**
    5.1 **Android**
    - Primary Target: Uses `flutter_whatsapp_stickers` with `contents.json`.
    - Storage: SAF ensures compatibility with Android 11+. Test thoroughly on Android 10, 11, 12, 13, 14+.
    5.2 **iOS (Future)**
    - Challenge: Requires `sticker_packs.wasticker`.
    - Options:
        - Export `.wastickers` files.
        - Research iOS-specific integration.

6.  **Constraints and Assumptions**
    6.1 **Constraints**
    - Static stickers only initially.
    - `contents.json` in documents directory due to dynamic generation.
    - Relies on user having WhatsApp installed and granting SAF permissions correctly.
    - App performance depends on device capabilities and number of stickers.
    6.2 **Assumptions**
    - WhatsApp accepts validated `contents.json` from documents directory via the integration package.
    - SAF provides reliable access to the sticker directory across target Android versions.
    - The `/Android/media/com.whatsapp/WhatsApp/Media/WhatsApp Stickers` path is relatively stable.
    - Users understand the concept of assigning emojis for search.

7.  **Success Metrics**
    - Functional: 95% of packs created successfully appear in WhatsApp.
    - Searchability: 100% of stickers with 1-3 emojis are findable in WhatsApp via assigned emojis.
    - Validation: 100% of generated `contents.json` files pass WhatsApp specs (based on integration success).
    - User Satisfaction: High rating/positive feedback regarding ease of use and searchability improvement.
    - Performance: App remains responsive (<5s interaction delays) even with 10k+ stickers.

8.  **Risks and Mitigations**
    - **Risk:** User skips or rushes emoji input, impacting searchability.
        - **Impact:** High
        - **Mitigation:** Enforce 1-3 emojis selection in UI/validation with clear mandatory indicators and tooltips.
    - **Risk:** WhatsApp changes `contents.json` guidelines or integration method.
        - **Impact:** Medium-High
        - **Mitigation:** Monitor WhatsApp developer documentation quarterly. Design metadata generation flexibly.
    - **Risk:** WhatsApp sticker directory path changes.
        - **Impact:** High
        - **Mitigation:** Allow users to manually re-select the directory if the cached URI fails. Monitor community forums/WhatsApp updates.
    - **Risk:** Performance degradation with very large (>50k) sticker libraries.
        - **Impact:** Medium
        - **Mitigation:** Thorough performance testing. Optimize file listing, image loading, and filtering algorithms. Consider database caching of sticker metadata if SAF listing is too slow.
    - **Risk:** SAF permission denied or URI becomes invalid.
        - **Impact:** High (core functionality blocked)
        - **Mitigation:** Implement robust error handling and clear user guidance to re-grant permission. Use persistent permissions where possible.

9.  **Testing Plan**
    - **Unit Tests:** Metadata validation (emoji count, field lengths, JSON structure), identifier generation, input validation logic.
    - **Integration Tests:** SAF directory access and file listing. Database operations (tags). Pack addition process mocking WhatsApp call.
    - **Manual/End-to-End Tests:** Full user flow on various Android versions (10-14+) and devices (low/mid/high-end). Test pack creation with min/max stickers. Verify emoji search in WhatsApp manually. Test with corrupted/invalid sticker files. Test filter combinations. Test edge cases (see section 11).

10. **Future Enhancements**
    - iOS: Generate `sticker_packs.wasticker` or use alternative integration.
    - Animated Stickers: Add `animated_sticker_pack` field support, handle APNG/animated WebP validation if needed.
    - Guideline Updates: Adapt to new WhatsApp metadata features.
    - Pack Editing: Allow users to add/remove stickers or edit metadata of existing packs created by the app.
    - Cloud Backup/Sync: Option to backup pack configurations.
    - Advanced Filtering: Filter by sticker dimensions, file size, dominant colors.

11. **Potential Edge Cases & Handling**
    - **SAF/Storage:**
        - *Denial:* User denies SAF permission. -> *Handling:* Show informative message explaining why permission is needed and prompt again.
        - *Wrong Folder:* User selects a non-WhatsApp sticker folder. -> *Handling:* Check for `.webp` files; if few/none found, warn user they might have selected the wrong folder.
        - *Empty Folder:* WhatsApp sticker folder exists but is empty. -> *Handling:* Display "No stickers found" message gracefully.
        - *Corrupt Files:* Directory contains corrupt `.webp` files. -> *Handling:* Skip corrupt files during listing/display, potentially log error. Inform user during pack creation if a selected file is corrupt.
        - *Permission Issues:* Individual sticker files have read errors. -> *Handling:* Skip files with errors, provide feedback if selected for a pack.
        - *Out of Space:* Device storage full during file copy or JSON generation. -> *Handling:* Catch I/O errors, inform user with "Not enough storage space" message.
        - *Invalid URI:* Cached SAF URI becomes invalid. -> *Handling:* Detect invalid URI on app start/access attempt, clear cache, prompt user to re-select folder.
    - **User Input/Interaction:**
        - *Invalid Chars:* User enters special characters in Name/Publisher not allowed by WhatsApp. -> *Handling:* Sanitize input or validate against allowed character sets if known; rely on WhatsApp integration to fail otherwise.
        - *Zero/Too Many Stickers:* User tries to create pack with 0 or >30 stickers. -> *Handling:* Disable creation button or show validation message.
        - *Creation Interrupt:* User backgrounds app during pack creation. -> *Handling:* Attempt to complete operation in background if possible (short tasks); otherwise, save state and resume/restart process gracefully, or cancel and inform user.
    - **Data Handling:**
        - *Boundary Validation:* Sticker size is exactly 100KB, or dimensions slightly off 512x512. -> *Handling:* Ensure validation logic strictly adheres to limits (<= 100KB, == 512x512).
        - *Tray Image Failure:* First selected sticker is corrupt/invalid for tray generation. -> *Handling:* Attempt using the next valid sticker, or use a default placeholder tray icon and inform the user.
        - *Emoji Support:* User selects very new/obscure emojis. -> *Handling:* Use standard emoji picker; WhatsApp search compatibility depends on WhatsApp itself.
    - **External Dependencies:**
        - *Integration Failure:* `flutter_whatsapp_stickers` fails. -> *Handling:* Catch errors from the package, display user-friendly message (e.g., "Could not add pack to WhatsApp. Is it installed?"). Do not delete the generated pack files in the app's directory so user can retry.
        - *WhatsApp Not Installed:* Integration attempt fails because WhatsApp isn't present. -> *Handling:* Check if WhatsApp is installed before attempting integration; inform user if missing.