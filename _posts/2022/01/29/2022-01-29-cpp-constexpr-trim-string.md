---
title: "C++ constexpr trim string"
categories:
  - blog
tags:
  - c++
  - constexpr
  - template
  - string
  - fmtlib
---
My team standardizes on C++14 at work. Fortunately, I am allowed to use some header libraries. I found handy libraries such as backward-cpp and fmt::fmtlib to help me print debug messages.

My code throw exception when anything bad happens. I follow [this](https://github.com/GPMueller/mwe-cpp-exception) example to throw and handled nested exception in C++11.

When I handle my exception, I use fmtlib, instead of cout, to display layers of messages. I come across [this](https://github.com/fmtlib/fmt/issues/1260) that shows me how to indent each level.

I frequently need to print some array of hex numbers. This is an example. No loop is required. I don't even need to use `fmt::join()` from `<fmt/ranges>` to print an array if I am fine with reading my numbers in dec.
```
#include <fmt/core.h>
#include <fmt/format.h>
#include <fmt/ranges.h>
#include <array>

std::array<unsigned int , 4> a{0x87654321, 0x89abcdef, 0xaa55aa55, 0x41424344};

int main() {
  fmt::print("Array 0x{:x}.", fmt::join(a, ", 0x"));
}
```
Try the [code](https://godbolt.org/z/EjYs5391x) on Compiler Explorer.

I put file name line number in my debug messages. Our build system passes the full pathname of the file to gcc. `__FILE__` becomes a long string. I used to trim the file name at runtime before I print the log message.

Recently, I found this [post](https://stackoverflow.com/questions/8487986/file-macro-shows-full-path). Andry's answer (on Jan 23 '19 at 21:05) shows that template meta programming can find the final '/' or '\' char in a string at compile-time. i.e. a constexpr strrchr() function.

You can try [Andry's answer](https://godbolt.org/z/u6s8j3) on Compiler Explorer.

Based on this technique, I wrote another constexpr function that looks for "::" from the right. Furthermore, it starts the search AFTER seeing the right '('. The end result of this would print the class and function name, trimming the namespaces on the left and the input arguments on the right.

```
template <typename T, size_t S>
inline constexpr size_t constexpr_strrchr(const char ch, const T (& str)[S], size_t i = S - 1)
{
    return (str[i] == ch)? i : (i > 0 ? constexpr_strrchr(ch, str, i - 1) : 0);
}

template <typename T>
inline constexpr size_t constexpr_strrchr(const char ch, T (& str)[1])
{
    return 0;
}


template <typename T, size_t S>
inline constexpr size_t get_open_bracket_offset(const T (& str)[S])
{
    size_t i = S - 1;
    const char ch = '(';

    // Return the exact location of the bracket. Don't add/substract.
    return constexpr_strrchr(ch, str, i);
}

template <typename T, size_t S>
inline constexpr size_t get_file_name_offset(const T (& str)[S])
{
    size_t i = S - 1;
    const char ch = '/';

    // Return the position AFTER the '/'. Add 1.
    return constexpr_strrchr(ch, str, i) + 1;
}

template <typename T, size_t S>
inline constexpr size_t get_class_func_offset(bool seen_open_bracket, unsigned int visible_colon, const T (& str)[S], size_t i = S - 1)
{
    seen_open_bracket = (str[i] == '(') ? true : seen_open_bracket;

    return (seen_open_bracket && (str[i] == ':')) ?
            ((visible_colon == 0)?  i + 1 : get_class_func_offset(seen_open_bracket, visible_colon - 1, str, i - 1))
        : (i > 0 ? get_class_func_offset(seen_open_bracket, visible_colon, str, i - 1) : 0);
}

// Construct a string_view at an offset that skips everything until the final '/'
#define CPP_FILE (fmt::string_view((__FILE__ + get_file_name_offset(__FILE__))))

// Construct a string_view at an offset that begins after the third ':', and end the string_view before the final '('
#define NUM_OF_COLON_TO_SKIP (2)
#define CPP_FUNC (fmt::string_view(__PRETTY_FUNCTION__ + get_class_func_offset(false, NUM_OF_COLON_TO_SKIP, __PRETTY_FUNCTION__), get_open_bracket_offset(__PRETTY_FUNCTION__) - get_class_func_offset(false, NUM_OF_COLON_TO_SKIP, __PRETTY_FUNCTION__)))

#define DBG_PREFIX                                                            \
    dbg_prefix(                                                          \
        CPP_FILE, __LINE__, CPP_FUNC  \
    )

inline std::string dbg_prefix(
    fmt::string_view file,
    unsigned int line,
    fmt::string_view func
) {
    return fmt::format("{}:{} {} ", file, line, func);
}
```
I plan to standardize my debug log to always include file name, line, class name and function name in all my debug messages.

You can try [my code](https://godbolt.org/z/dP1jzaf7T) on Compiler Explorer.

I experimented with C++17 string_view. I hoped it can avoid messy template meta programming. I tested on Compiler Explorer but the function that extracts class name and function name ended up as a regular function, not an inline constexpr function. Here is the failed [code](https://godbolt.org/z/597637f6Y)

By the way, I came across this [post](https://davidgorski.ca/posts/truncate-string-whitespace-compiletime-cpp/) showing a technique to trim whitespaces at compile-time. I did not it but it might be useful in the future.

