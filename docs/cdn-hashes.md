---
title: CDN Hashes
description: Explains how to get the CDN subdomain ID from the CDN hash.
---

# CDN hashes
Some endpoints, like the imageUrl provided by `thumbnails.roblox.com/v1/users/avatar-3d?userId=1`, don't provide a full
CDN URL and only provide raw hashes, like this: `bbdb80c2b573bf222da3e92f5f148330`.
We need to turn this into a full CDN URL. 
A CDN URL looks like `t[X].rbxcdn.com/bbdb80c2b573bf222da3e92f5f148330` where X is the CDN number.
The CDN number ranges from 0 to 7, so you might be tempted to send a request to t0, then t1, and keep going until you
reach the one containing the object. This works, but it's quite wasteful as you send up to 8 requests for one object.

<div id="hash-calculator-box">
<button class="md-clipboard md-icon" id="hash-calculator-copy-button" title="Copy to clipboard" data-clipboard-text=""></button>
<span id="hash-calculator-contents">
https://<select id="hash-calculator-select">
<option value="t">t</option>
<option value="c">c</option>
</select><span id="hash-calculator-num">X</span>.rbxcdn.com/<input type="text" autocorrect="off" id="hash-calculator-input" maxlength="32" placeholder="00000000000000000000000000000000">
</span>
</div>

<script>
const hashInput = document.getElementById("hash-calculator-input")
const hashNum = document.getElementById("hash-calculator-num")
const hashCdnSelector = document.getElementById("hash-calculator-select")
const hashCopyButton = document.getElementById("hash-calculator-copy-button")

const getCdnUrl = (cdnType, hash) => {
    const t = [...hash].reduce((lastCode, char) => lastCode ^ char.charCodeAt(0), 31)

    return `https://${cdnType}${t % 8}.rbxcdn.com/${hash}`;
}

const update = () => {
    const hash = hashInput.value
    const hashNumValue = ([...hash].reduce((lastCode, char) => lastCode ^ char.charCodeAt(0), 31)) % 8
    hashNum.textContent = hashNumValue
    hashCopyButton.setAttribute("data-clipboard-text", getCdnUrl(hashCdnSelector.value, hash))
}

update();

hashInput.addEventListener("input", update);
</script>

A common implementation is as follows:  
Create a function that takes in a string `hash`. In this function, define `x` to `31`.
Next, we loop through the first 32 characters in `hash`, and in each iteration set the `x` variable to itself bitwise 
XORed against the integer representation of that character (`x ^= chr`). Then we return `x`.

We can then use the `x` variable to compose the final URL with the pattern `https://t{x}.rbxcdn.com/{hash}`.

Here are some test cases for this function:
```
func("bbdb80c2b573bf222da3e92f5f148330") -> 5
func("139602eb7c640c43833470e07caada4a") -> 7
func("b717c50234c3d91b0be7dbfc9c588ed4") -> 0
```

## Examples

=== "Python"
    ```py
    def get_cdn_url(hash):
    i = 31
    for char in hash[:32]:
        i ^= ord(char)  # i ^= int(char, 16) also works
    return f"https://t{i%8}.rbxcdn.com/{hash}"

    # alternatively:
    from functools import reduce
    
    def get_cdn_url(hash):
        t = reduce(lambda last_code, char: last_code ^ ord(char), hash, 31)
        
        return f"https://t{t % 8}.rbxcdn.com/{hash}"
    ```
=== "Golang"
    ```go
	package pkg

	import "fmt"

	// GetCdnUrl
	func GetCdnUrl(hash string) string {
		if hash == "" {
			panic("hash is empty")
		}

		var i int = 31

		for _, char := range hash {
			i = i ^ int(char)
		}

		return fmt.Sprintf("https://t%d.rbxcdn.com/%s", i%8, hash)
	}
    ```
=== "Elixir"
    ```elixir
    defmodule CDN do
      @spec get_cdn_url(String.t()) :: String.t()
      def get_cdn_url(hash) do
        t = hash
        |> String.to_charlist
        |> Enum.reduce(31, fn char, last_code -> Bitwise.bxor(last_code, char) end)

        "https://t#{rem(t, 8)}.rbxcdn.com/#{hash}"
      end
    end
    ```
=== "64 bit NASM Assembly"
    ```nasm
    ; nasm -felf64 cdn_hash.asm -o cdn_hash.o
    ; gcc -m64 -o cdn_hash cdn_hash.o -no-pie
    ; ./cdn_hash
    extern printf, snprintf

    section .text
        global main

    get_cdn_url:
        push rdi
        push rsi
        push rdx
        push rcx
        push r8
        push rax

        ; rdi is the accumulator
        mov rdi, 31

        jmp .is_at_end

    .xor_t:
        push rax
        ; rax is 8 bytes, get the first byte by AND'ing it by 255
        mov rax, [rax]
        and rax, 0xFF
        xor rdi, rax
        pop rax

        ; increment hash pointer
        inc rax

    .is_at_end:
        cmp byte[rax], 0
        jne .xor_t

    .fmt_cdn_url:
        mov rax, rdi
        xor rdx, rdx
        mov rsi, 8
        div rsi

        ; t
        mov rcx, rdx
        ; buffer
        lea rdi, [cdn_url]
        ; buffer size
        mov rsi, 55
        ; format
        lea rdx, [url_fmt]
        ; hash
        pop rax
        mov r8, rax
        push rax
        xor rax, rax
        call snprintf

        pop rax
        pop r8
        pop rcx
        pop rdx
        pop rsi
        pop rdi

        ret

    main:
        lea rax, [hash]
        call get_cdn_url

        lea rdi, [s_fmt]
        lea rsi, [cdn_url]
        xor rax, rax
        call printf

        mov rax, 60
        mov rdi, 0
        syscall

    section .data
        cdn_url: times 55 db 0
        s_fmt: db "%s", 10, 0
        url_fmt: db "https://t%d.rbxcdn.com/%s", 0
        hash: db "bbdb80c2b573bf222da3e92f5f148330", 0

    ```
=== "JavaScript"
    ```js
    const getCdnUrl = (hash) => {
        const t = [...hash].reduce((lastCode, char) => lastCode ^ char.charCodeAt(0), 31)
    
        return `https://t${t % 8}.rbxcdn.com/${hash}`;
    }
    ```
=== "C#"
    ```csharp
    using System;
    using System.Linq;
    
    string GetCdnUrl(string hash) {
         int t =  hash.ToCharArray().Aggregate(31, (lastCode, character) => lastCode ^ (int)character);
     
         return $"https://t{t % 8}.rbxcdn.com/{hash}";
    }
    ```
=== "Ruby"
    ```ruby
    def get_cdn_url(hash)
      t = hash.codepoints.reduce(31) { |last_code, code| last_code ^ code }
      "https://t#{t % 8}.rbxcdn.com/#{hash}"
    end
    ```
=== "C++"
    ```cpp
    std::string getCdnUrl(const std::string& hash)
    {
        if (hash.empty()) throw std::exception("Hash cannot be empty");
    
        int i = 31;
    
        for (char const& c : hash)
        {
            i ^= (int)c;
        }
    
        char buff[100];
        snprintf(buff, sizeof(buff), "https://t%d.rbxcdn.com/%s", i % 8, hash.c_str());
    
        return std::string(buff);
    }
    ```
=== "C"
    ```c
    void getCdnUrl(char *hash, char *buffer) {
        int i = 31;
        int hashLength = strlen(hash);
        for (int j = 0; j < hashLength; j++) {
            i ^= (int)hash[j];
        }
    
        snprintf(buffer, 55, "https://t%d.rbxcdn.com/%s", i % 8, hash);
    }
    ```
=== "Rust"
    ```rust
    fn get_cdn_url(hash: &str) -> String {
        let t = hash.as_bytes().iter().fold(31, |last_code, code| {
            last_code ^ code
        });
        
        format!("https://t{}.rbxcdn.com/{}", t % 8, hash)
    }
    ```
=== "Lua 5.3"
    ```lua
    local function getCdnUrl(hash)
        local i = 31
        for _, code in utf8.codes(hash) do
            i = i ~ code
        end
    
        return string.format("https://t%d.rbxcdn.com/%s", i % 8, hash)
    end
    ```
=== "Lua 5.2/Luau"
    ```lua
    local function getCdnUrl(hash)
        local i = 31
        for _, code in utf8.codes(hash) do
            i = bit32.bxor(i, code)
        end
    
        return string.format("https://t%d.rbxcdn.com/%s", i % 8, hash)
    end
    ```
=== "Java"
    ```java
    String getCdnUrl(String hash) {
        int i = 31;
        for (char character : hash.toCharArray()) {
            i ^= (int) character;
        }
            
        return String.format("https://t%d.rbxcdn.com/%s", i % 8, hash);
    }
    ```
=== "Kotlin"
    ```kotlin
    fun getCdnUrl(hash: String): String {
        var i = 31
        hash.forEach({ character: Char ->
            i = i xor character.toInt()
        });
        
        return "https://t${i % 8}.rbxcdn.com/${hash}"
    }
    ```
=== "Crystal"
    ```crystal
    def get_cdn_url(hash : String): String
      t = hash.codepoints.reduce(31) { |last_code, code| last_code ^ code }
      "https://t#{t % 8}.rbxcdn.com/#{hash}"
    end
    
    ```
=== "F#"
    ```fsharp
    let getCdnUrl (hash: string) =
    let t = hash |> Seq.fold (fun lastCode char -> lastCode ^^^ (int)char) 31
    
    $"https://t{t % 8}.rbxcdn.com/{hash}"
    ```
=== "PowerShell"
    ```powershell
    function get-cdn-url {
        param (
            [string] $hash
        )
        
        if ([string]::IsNullOrEmpty($hash)) { throw [System.ArgumentNullException]::new("hash"); }
    
        [int] $i = 31;
    
        foreach ($c in $hash.ToCharArray()) {
            $i = $i -bxor $c;
        }
    
        return "https://t$($i % 8).rbxcdn.com/$($hash)"
    }
    ```
