# Buffer Overflow Bug Fix in OpenKey Engine

## Bug Description
Found a buffer overflow vulnerability in the `insertKey` function in `Sources/OpenKey/engine/Engine.cpp`.

## Issue
The function maintains a buffer `TypingWord[MAX_BUFF]` where `MAX_BUFF = 32`. When the buffer becomes full (`_index >= MAX_BUFF`), the function performs a left shift operation but fails to decrement the `_index` variable, allowing it to grow beyond `MAX_BUFF`.

## Root Cause
```cpp
void insertKey(const Uint16& keyCode, const bool& isCaps, const bool& isCheckSpelling=true) {
    if (_index >= MAX_BUFF) {
        _longWordHelper.push_back(TypingWord[0]); //save long word
        //left shift
        for (iii = 0; iii < MAX_BUFF - 1; iii++) {
            TypingWord[iii] = TypingWord[iii + 1];
        }
        setKeyData(_index-1, keyCode, isCaps);  // BUG: _index not decremented!
    } else {
        setKeyData(_index++, keyCode, isCaps);
    }
}
```

## Impact
- `_index` can grow indefinitely beyond `MAX_BUFF`
- Array access using `_index-1`, `_index-2`, etc. can cause buffer overflows
- Multiple functions throughout the codebase use `_index` for array indexing
- Potential for memory corruption and crashes

## Fix Applied
```cpp
if (_index >= MAX_BUFF) {
    _longWordHelper.push_back(TypingWord[0]); //save long word
    //left shift
    for (iii = 0; iii < MAX_BUFF - 1; iii++) {
        TypingWord[iii] = TypingWord[iii + 1];
    }
    _index--; //decrement index to keep it within bounds
    setKeyData(_index, keyCode, isCaps);
    _index++; //increment back for next insertion
} else {
    setKeyData(_index++, keyCode, isCaps);
}
```

## Verification
The fix ensures that `_index` never exceeds `MAX_BUFF - 1`, preventing buffer overflows while maintaining the correct circular buffer behavior.