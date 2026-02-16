# TinFS

**TinFS Is Not File System**

A clean, logical file hierarchy for Android that actually makes sense.

> ⚠️ **Work in Progress** - Currently in concept/planning stage

---

## The Problem

Android's filesystem is chaos:

```
/ (root)
├── [30+ confusing directories]
├── acct/ apex/ bin@ bootstrap-apex/ bugreports@ cache/
├── config/ d@ data/ data_mirror/ debug_ramdisk/ dev/
├── init@ linkerconfig/ metadata/ mnt/ odm/ odm_dlkm/
├── oem/ postinstall/ proc/ product/ sdcard@ storage/
├── sys/ system/ system_dlkm/ system_ext/ tmp/
├── vendor/ vendor_dlkm/ lost+found/
└── ...nobody knows what half of these are
```

**App data scattered everywhere:**
```
com.telegram.messenger lives in:
├── /data/app/~~random~~/com.telegram.messenger/     (APK)
├── /data/data/com.telegram.messenger/                (app data)
├── /sdcard/Android/data/com.telegram.messenger/      (primary storage)
└── /sdcard/Android/obb/com.telegram.messenger/       (game files)
```

**User files mixed with garbage:**
```
/sdcard/
├── DCIM/
├── Download/
├── Android/ (technical junk)
├── .thumbnails/ (junk)
├── [random app folders everywhere]
└── ...good luck finding your files
```

---

## The Solution

TinFS reorganizes everything using symlinks - **without breaking compatibility:**

### New Clean Structure

```
/
├── system/          # System partition (untouched)
├── vendor/          # Vendor partition (untouched)
├── data/            # Data partition (untouched)
├── proc/            # Virtual FS (untouched)
├── sys/             # Virtual FS (untouched)
├── dev/             # Virtual FS (untouched)
│
├── app/             # ← NEW: All applications
│   ├── sys/         #    System apps (symlinks to /system/app, /vendor/app)
│   └── user/        #    User apps - EVERYTHING in one place
│       └── com.telegram.messenger/
│           ├── apk/      → real data (moved from /data/app)
│           ├── data/     → real data (moved from /data/data)
│           ├── storage/  → real data (moved from /sdcard/Android/data)
│           └── obb/      → real data (moved from /sdcard/Android/obb)
│
├── home/            # ← NEW: User files
│   └── user/
│       ├── downloads/
│       ├── photos/
│       ├── music/
│       ├── videos/
│       └── documents/
│
├── cache/           # ← NEW: All cache
│   └── system/      → system cache
│
└── legacy/          # ← NEW: Old paths (symlinks for compatibility)
    ├── data/
    ├── storage/
    └── [symlinks to maintain compatibility]
```

### How Data is Organized

**Before TinFS:**
- APK: `/data/app/com.telegram.messenger/`
- Data: `/data/data/com.telegram.messenger/`
- Storage: `/sdcard/Android/data/com.telegram.messenger/`
- OBB: `/sdcard/Android/obb/com.telegram.messenger/`

**After TinFS:**
```
/app/user/com.telegram.messenger/
├── apk/      (real data here)
├── data/     (real data here)
├── storage/  (real data here)
└── obb/      (real data here)
```

Old paths become symlinks:
```
/data/data/com.telegram.messenger/ → /app/user/com.telegram.messenger/data/
/data/app/com.telegram.messenger/  → /app/user/com.telegram.messenger/apk/
```

**Apps work normally, users see clean structure!** ✨
 
---

## How It Works

**TinFS doesn't modify the filesystem or Android code.**

Instead:
1. Real data lives in new clean structure (`/app/user/`, `/home/user/`)
2. Old Android paths become symlinks to new locations
3. Apps access data through symlinks → everything works
4. Users see clean, logical organization

**This Is Not FS** - it's a symlink layer providing a clean interface over the existing mess.

---

## About Root Directory

**Q: Won't root still have many symlinks?**

**A:** Yes, for compatibility. But:
- **Regular users** never browse root - they use file managers that open `/home/user/`
- **Power users** understand why symlinks exist (compatibility)
- **Developers** work with clean `/app/user/` structure

The important thing: **your data is organized cleanly**, even if root has legacy symlinks.

Future versions may include:
- File manager modifications to hide legacy paths
- UI-level hiding of compatibility layer

---

## Installation

🚧 Not ready yet - still in planning/design phase

---


## Contributing

This is early stage! Feedback and ideas welcome.

**How to help:**
- Open issues with suggestions
- Test on your device (when ready)
- Report compatibility problems
- Contribute to documentation

---

## FAQ

**Q: Will this break my apps?**
A: No. Apps use old paths which are symlinked to new structure. Everything works.

**Q: Can I uninstall?**
A: Yes. Remove symlinks, move data back. (We'll provide an uninstaller)

**Q: Does this work with OTA updates?**
A: Not tested yet. Likely needs reinstallation after major updates.

**Q: Why not just patch Android?**
A: This works on ANY ROM without recompiling. Much more accessible.

**Q: What about SELinux?**
A: Symlinks inherit correct security contexts. Should work fine.

---

## License

**GNU General Public License v3.0**

This code stays free forever. If you use it, share your improvements.

---

## Author

**Created by [sayavc](https://github.com/sayavc)**

*Conceived in a bathtub, born from frustration with Android's filesystem chaos.*

*Inspired by the elegance of Unix FHS and the philosophy of GNU.*

```
