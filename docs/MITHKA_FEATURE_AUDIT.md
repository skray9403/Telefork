# Mithka-inspired capability audit

This audit compares the areas discussed for Mithka with the existing Telefork/Forkgram implementation. The intent is to reuse the established native Telegram architecture rather than introduce a second client engine.

## Implemented in this update

### Privacy-preserving Telegram device identity

Telegram connection initialization in `ConnectionsManager` previously sent `Build.MANUFACTURER + Build.MODEL`. Telefork now supports three choices in **Fork Client Settings → Telegram device identity**:

- **Generic Android** (default): reports `Android`.
- **Actual device model**: reports the manufacturer and model supplied by Android.
- **Custom device name**: reports a user-entered value.

The setting is global across local accounts. It is read when a `ConnectionsManager` instance initializes, so a restart is required after changing it. This changes the client-supplied device-model string in Telegram session details; it does not make the account or network connection anonymous.

## Existing capabilities retained from the Telegram codebase

### Multiple accounts and session isolation

- `UserConfig.MAX_ACCOUNT_COUNT` is 90.
- Account configuration is stored per account.
- `ConnectionsManager`, `MessagesStorage`, and `FileLoader` use per-account instance arrays.
- Account-specific connection and storage directories are maintained by the existing implementation.

Relevant source:
- [UserConfig.java](../TMessagesProj/src/main/java/org/telegram/messenger/UserConfig.java)
- [ConnectionsManager.java](../TMessagesProj/src/main/java/org/telegram/tgnet/ConnectionsManager.java)
- [MessagesStorage.java](../TMessagesProj/src/main/java/org/telegram/messenger/MessagesStorage.java)
- [FileLoader.java](../TMessagesProj/src/main/java/org/telegram/messenger/FileLoader.java)

### Chat and media caching

- Message and chat data are stored in the existing SQLite cache database, configured for WAL mode.
- `ImageLoader` already maintains bounded LRU caches for full-size and small images, plus specialized animated media caches.
- File loading uses dedicated small/large queues and per-account loader instances.

Relevant source:
- [MessagesStorage.java](../TMessagesProj/src/main/java/org/telegram/messenger/MessagesStorage.java)
- [ImageLoader.java](../TMessagesProj/src/main/java/org/telegram/messenger/ImageLoader.java)
- [FileLoader.java](../TMessagesProj/src/main/java/org/telegram/messenger/FileLoader.java)

### Download management and storage paths

- `DownloadController` maintains separate auto-download presets for contacts, private chats, groups, and channels.
- Photo, video, audio, and document downloads have separate queues and policy masks.
- `FileLoader` resolves media directories through the existing `FileLoader.setMediaDirs` path setup.

Relevant source:
- [DownloadController.java](../TMessagesProj/src/main/java/org/telegram/messenger/DownloadController.java)
- [FileLoader.java](../TMessagesProj/src/main/java/org/telegram/messenger/FileLoader.java)
- [ImageLoader.java](../TMessagesProj/src/main/java/org/telegram/messenger/ImageLoader.java)

### Message navigation and search

- The Android client already uses `ActionBarLayout` and `INavigationLayout` fragment stacks for push/pop navigation.
- `LaunchActivity` maintains the main fragment stack and participates in Android back navigation, including predictive-back handling.
- Chat and dialog search are part of the existing native UI.

Relevant source:
- [LaunchActivity.java](../TMessagesProj/src/main/java/org/telegram/ui/LaunchActivity.java)
- [ActionBarLayout.java](../TMessagesProj/src/main/java/org/telegram/ui/ActionBar/ActionBarLayout.java)
- [INavigationLayout.java](../TMessagesProj/src/main/java/org/telegram/ui/ActionBar/INavigationLayout.java)
- [DialogsActivity.java](../TMessagesProj/src/main/java/org/telegram/ui/DialogsActivity.java)
- [ChatActivity.java](../TMessagesProj/src/main/java/org/telegram/ui/ChatActivity.java)

### Native Android integration and performance

Telefork is already based on Telegram's native Android Java/Kotlin UI and JNI/C++ networking/media stack. Its current image caches, SQLite storage, and queued file operations are established performance infrastructure. Mithka's Flutter/TDLib architecture is therefore not a suitable subsystem to transplant into this project.

Relevant source:
- [ConnectionsManager.java](../TMessagesProj/src/main/java/org/telegram/tgnet/ConnectionsManager.java)
- [ConnectionsManager.cpp](../TMessagesProj/jni/tgnet/ConnectionsManager.cpp)
- [ImageLoader.java](../TMessagesProj/src/main/java/org/telegram/messenger/ImageLoader.java)

## Deliberately not duplicated

The account manager, message database, media cache, downloader, and navigation stack are already present. Replacing them with parallel Mithka-inspired implementations would increase regression risk without adding the capability being discussed. Future work should target measured gaps (for example, a new download-location UI or cache tuning backed by profiling) rather than adding duplicate infrastructure.
