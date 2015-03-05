---
layout: post
title: "zend buildin functions"
date: 2015-03-03 19:40:13 +0800
comments: true
categories: 
---

头文件
> zend_buildin_functions.h

主要定义了函数
``` c
int zend_startup_builtin_functions(void);

```

<!-- more -->
具体实现如下：

``` c
zend_module_entry zend_builtin_module = {
    STANDARD_MODULE_HEADER,
    "Core",
    builtin_functions,
    ZEND_MINIT(core),
    NULL,
    NULL,
    NULL,
    NULL,
    ZEND_VERSION,
    STANDARD_MODULE_PROPERTIES
};

int zend_startup_builtin_functions(void)
{
    zend_builtin_module.module_number = 0;
    zend_builtin_module.type = MODULE_PERSISTENT;
    return (EG(current_module) = zend_register_module_ex(&zend_builtin_module)) == NULL ? FAILURE : SUCCESS;
}

```
这段代码具体指什么，稍后分析

### 2 函数定义过程

> zend_version(); 
``` c
static ZEND_FUNCTION(zend_version);

static const zend_function_entry builtin_functions[] = {
    ZEND_FE(zend_version,       arginfo_zend__void)
    ···
};

/* proto string zend_version(void)
   Get the version of the Zend Engine */
ZEND_FUNCTION(zend_version)
{
    RETURN_STRINGL(ZEND_VERSION, sizeof(ZEND_VERSION)-1);
}

```

ZEND_FUNCTION 在zend_API.h中定义，具体如下：
``` c
#define ZEND_FN(name) zif_##name
#define ZEND_NAMED_FUNCTION(name)       void name(INTERNAL_FUNCTION_PARAMETERS)
#define ZEND_FUNCTION(name)             ZEND_NAMED_FUNCTION(ZEND_FN(name))

```
INTERNAL_FUNCTION_PARAMETERS是在zend.h中定义的，表示函数的执行参数：
``` c
#define INTERNAL_FUNCTION_PARAMETERS zend_execute_data *execute_data, zval *return_value
```

所以ZEND_FUNCTION(zend_version)展开后便是：

``` c
void zif_zend_version(zend_execute_data *execute_data, zval *return_value)
```

###3 

zend_register_module_ex

``` 
ZEND_API zend_module_entry* zend_register_module_ex(zend_module_entry *module)
{
    size_t name_len;
    zend_string *lcname;
    zend_module_entry *module_ptr;

    if (!module) {
        return NULL;
    }

    /* Check module dependencies */
    if (module->deps) {
        const zend_module_dep *dep = module->deps;

        while (dep->name) {
            if (dep->type == MODULE_DEP_CONFLICTS) {
                name_len = strlen(dep->name);
                lcname = zend_string_alloc(name_len, 0);
                zend_str_tolower_copy(lcname->val, dep->name, name_len);

                if (zend_hash_exists(&module_registry, lcname)) {
                    zend_string_free(lcname);
                    /* TODO: Check version relationship */
                    zend_error(E_CORE_WARNING, "Cannot load module '%s' because conflicting module '%s' is already loaded", module->name, dep->name);
                    return NULL;
                }
                zend_string_free(lcname);
            }
            ++dep;
        }
    }

    name_len = strlen(module->name);
    lcname = zend_string_alloc(name_len, 1);
    zend_str_tolower_copy(lcname->val, module->name, name_len);

    if ((module_ptr = zend_hash_add_mem(&module_registry, lcname, module, sizeof(zend_module_entry))) == NULL) {
        zend_error(E_CORE_WARNING, "Module '%s' already loaded", module->name);
        zend_string_release(lcname);
        return NULL;
    }
    zend_string_release(lcname);
    module = module_ptr;
    EG(current_module) = module;

    if (module->functions && zend_register_functions(NULL, module->functions, NULL, module->type)==FAILURE) {
        EG(current_module) = NULL;
        zend_error(E_CORE_WARNING,"%s: Unable to register functions, unable to load", module->name);
        return NULL;
    }

    EG(current_module) = NULL;
    return module;
}

```