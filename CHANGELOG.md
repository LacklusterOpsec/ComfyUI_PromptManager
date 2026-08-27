# Changelog

All notable changes to this project are documented in this file.

## [Unreleased] - 2026-08-27

### 🐛 Bug Fixes & Resiliency

- **Image Monitor Fallback Dictionary Parsing**: Fixed `KeyError: 0` in `utils/image_monitor.py:L383-395` where `_get_fallback_prompt()` improperly indexed `get_recent_prompts(limit=1)` dictionary as a list (`recent_prompts[0]` $\rightarrow$ `recent_data.get("prompts", [])[0]`).
- **Secondary LoRA Hash Stripping**: Enhanced `_find_prompt_from_metadata()` in `utils/image_monitor.py` to attempt a stripped prompt lookup without `<lora:...>` tags when the exact combined prompt hash does not match an existing record in SQLite.
- **Dual Node Prompt Overwrite Protection**: Fixed `patched_save_images` in `utils/comfyui_integration.py:L240-270` to match thread-local prompts per node rather than overwriting all `PromptManager` nodes with the single last-executed prompt string (preserving distinct positive and negative prompts in workflow metadata).
- **Non-PNG Metadata Extraction**: Extended `extract_metadata()` in `utils/metadata_extractor.py:L88-145` to check `image.info` alongside `image.text`, enabling workflow and prompt metadata extraction for WebP, JPEG, and GIF formats.
- **Node Input Defaults**: Added `"default": ""` to `prepend_text` and `append_text` in `PromptManager.INPUT_TYPES` (`prompt_manager.py:L85-95`) to prevent missing key errors when optional inputs are omitted.
- **ComfyUI Dynamic Kwargs Acceptance**: Added `**kwargs` to execution functions across nodes:
  - `PromptManager.encode_prompt()` in `prompt_manager.py`
  - `PromptManagerText.process_text()` in `prompt_manager_text.py`
  - `PromptSearchList.search()` and `PromptSearchList.IS_CHANGED()` in `prompt_search_list.py`
- **Output Slot Naming**: Declared explicit `RETURN_NAMES = ("CONDITIONING", "STRING")` on `PromptManager` and `RETURN_NAMES = ("STRING",)` on `PromptManagerText` conforming to ComfyUI slot conventions.
- **Windows SQLite File Lock Cleanup**: Added `close()` proxy method to `PromptDatabase` in `database/operations.py` forwarding to `PromptModel.close()`, resolving `PermissionError: [WinError 32]` during database connection teardowns on Windows.
- **Cross-Platform Path Normalization**:
  - Added `os.path.normpath` to `_resolve_db_path()` in `database/operations.py` and `utils/diagnostics.py`.
  - Normalized subfolder paths and ancestor path segments to forward slashes in `get_prompt_subfolders()` for cross-platform API consistency.

### 🧪 Test Suite & CI Enhancements

- Updated test teardown methods across `tests/test_basic.py`, `tests/test_database.py`, `tests/test_lora_database.py`, `tests/test_diagnostics_config.py`, `tests/test_db_path_config.py`, and `tests/test_api.py` to close active database connections and temporary files before unlinking.
- Fixed patch targets in `tests/test_empty_text_handling.py` (`get_prompt_database` instead of `PromptDatabase`), `tests/test_diagnostics_config.py`, `tests/test_db_path_config.py`, and `tests/test_lora_utils.py` to match the refactored `promptmanager_core` package layout.
- Verified 100% pass rate across all 409 unit and integration tests on Python 3.13.

### 📝 Documentation
- Created `issue.txt` documenting all 14 architectural, runtime, and schema findings from the code review audit.
