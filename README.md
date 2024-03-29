## nanoJSONc - Event-Driven JSON Parser for C

[![License](https://img.shields.io/badge/License-BSD_3--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)
[![CMake on multiple platforms](https://github.com/saadshams/nanojsonc/actions/workflows/cmake-multi-platform.yml/badge.svg)](https://github.com/saadshams/nanojsonc/actions/workflows/cmake-multi-platform.yml)

nanoJSONc is a C-based event-driven architecture library for JSON parsing,
making it easy to integrate with your project's logic.

Users can register callback functions to respond to specific events, such as
key-value pairs within JSON objects or indices-values within JSON arrays, offering a
highly customizable and efficient approach to JSON parsing. nanoJSONc is ideal
for applications where tailored event handling is crucial to processing JSON
data effectively.

## Features
- Event-Driven Architecture
- Lightweight - ~250 lines of code
- Robust—almost "crash proof"
- No Dependencies
- Memory-Efficient - No `malloc(3)`
- Simple API, Pure Functions, No side-effects
- Parses deeply nested & mixed JSON objects and arrays (See [examples](https://github.com/saadshams/nanojsonc/tree/main/example))

## Installation
```commandline
vcpkg install nanojsonc
```

## Usage

### Parsing JSON Object

```c
struct Person { 
    char *name; 
    int age; 
};
static int callback(const char *const error, const char *const key, const char *const value, const char *const parentKey, void *object) {
    if (error) {
        fprintf(stderr, "Error: %s\n", error);
        return 1;
    }

    struct Person *person = object;
    if (strcmp(key, "name") == 0) (*person).name = strdup(value);
    if (strcmp(key, "age") == 0) (*person).age = atoi(value);
    return 0;
}

int main(void) {
    char *json = "{\"name\": \"John Doe\", \"age\": 25}";
    
    struct Person person = (struct Person){0};
    nanojsonc_parse_object(json, callback, NULL, &person, NULL);
    printf("Name: %s, Age: %d", person.name, person.age); // Name: John Doe, Age: 25
    
    free(person.name);
    return 0;
}
```

### Parsing JSON Array
```c
struct Hobby {
    char *name;
    struct Hobby *next;
};

static int callback(const char *const error, const char *const key, const char *const value, const char *const parentKey, void *object) {
    if (error) {
        fprintf(stderr, "Error: %s\n", error);
        return 1;
    }

    struct Hobby **hobbies = object;
    
    struct Hobby *hobby = malloc(sizeof(struct Hobby));
    hobby->name = strdup(value);
    hobby->next = NULL;

    struct Hobby **cursor;
    for (cursor = hobbies; *cursor; cursor = &(*cursor)->next);
    
    *cursor = hobby;

    return 0;
}

int main(void) {
    char *json = "[\"Reading\", \"Hiking\", \"Cooking\"]";
    
    struct Hobby *hobbies = NULL;
    nanojsonc_parse_array(json, callback, "hobbies", &hobbies);

    for (struct Hobby **cursor = &hobbies; *cursor; cursor = &(*cursor)->next)
        printf("%s ", (*cursor)->name); // Reading Hiking Cooking 
    
    return 0;
}
```

### Serializing JSON (Vanilla C)

```c
struct Person {
    char *name;
    int age;
};

int main() {
    struct Person *person = malloc(sizeof(struct Person));
    person->name = strdup("John");
    person->age = 25;
    
    // Example 1
    char *json = malloc(sizeof(char *));
    asprintf(&json, "{\n  \"name\": \"%s\",\n  \"age\": %d\n}", 
             person->name, person->age);
    printf("%s", json); // or send(clientSocket, json, strlen(json), 0);
    free(json);
    
    // Example 2 - Build incrementally
    char *json2 = NULL;
    size_t len = 0;
    
    len += asprintf(&json2, "{\n"); // Append incrementally
    len += asprintf(&json2, "%s  \"name\": \"%s\",\n", json2, person->name);
    len += asprintf(&json2, "%s  \"age\": %d\n", json2, person->age);
    len += asprintf(&json2, "%s}", json2);
    printf("%s", json2); // or send(clientSocket, json2, len, 0);
    free(json2);
    
    // Example 3 - Print to screen
    printf("{\n");
    printf("  \"name\": \"%s\",\n", person->name);
    printf("  \"age\": %d\n", person->age);
    printf("}");
    
    free(person->name);
    free(person);
}
```

## Examples

For a complete demo of how to use nanoJSONc, please refer to the [example](https://github.com/saadshams/nanojsonc/tree/main/example) directory.

* [API Docs](https://github.com/saadshams/nanojsonc/blob/main/include/parse.h)
* [Unit Tests](https://github.com/saadshams/nanojsonc/blob/main/test/test_screenshot.png)

## Status

Production - [Version 1.0.0](https://github.com/saadshams/nanojsonc/blob/master/VERSION)

## License

BSD 3-Clause License

* Copyright © 2023 [Saad Shams](https://www.linkedin.com/in/muizz/)
* Copyright © 2023 [Open Source Patterns Inc.]()

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
