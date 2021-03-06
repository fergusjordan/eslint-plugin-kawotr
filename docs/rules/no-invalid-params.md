# Enforce having a proper value fro `kawo.tr`

This rule ensures that the `cn` value passed to `kawo.tr` is not `null`, `false`, `undefined`, `''` or the same as the `en` parameter
so we don't miss translations in production.

# Fail

```javascript
kawo.tr( 'Test' )
```

```javascript
kawo.tr( 'Test', '' )
```

```javascript
kawo.tr( 'Test', 'Test' )
```

```javascript
kawo.tr( 'Test', '中文中文中文' )
```

```javascript
kawo.tr( 'Test', false, true )
```

```javascript
kawo.tr( 'Test', null, true )
```

# Pass

```javascript
kawo.tr( 'Test', '测试' )
```

```javascript
kawo.tr( 'Test', '测试', true )
```

Single, commonly used English strings can pass if a lookup function is provided that returns a valid value based on the `en` key.

```javascript
kawo.tr( 'Weibo' )
```

KAWO's lookup function for common string translations is something like this and passed through to the test's settings:

```javascript
export default function( lookup, returnLang ) {

    var localeObject,
        translations,
        translationsLowerCase = {};

    translations = {
        'weibo': {
            cn: '微博'
        }
    };

    for ( const key in translations )
        translationsLowerCase[ key.toLowerCase() ] = translations[ key ];

    // FIND IN STRINGS OBJECT
    localeObject = translationsLowerCase[ lookup.toLowerCase() ];

    // GET STRING › DEFAULT TO CHINESE RETURN LANGUAGE
    return ( localeObject && localeObject[ returnLang || 'cn' ] ) || false;
}
```

# Generously pass 

We give ourselves the benefit of the doubt in a few instances since we can't know certain variable values,
can't process VirtualDom nodes, etc.

We let through: 
- Variables (Identifier)
- Arrays (ArrayExpression) -- usually to pass in VirtualDom nodes like `h( 'strong' )`
- Conditionals (ConditionalExpression)
- Functions (CallExpression)
- Objects (MemberExpression)
- Ternary operators (LogicalExpression)
- Binary expressions (BinaryExpression) -- concatenated strings count as a binary expression
  - Note that we will try and treat as a string by running `eval( string )` in an attempt to perform the concatenation to run against our lookup function.

```javascript
kawo.tr( [ 'This post will be published ', h( 'strong', 'immediately' ) ] )
```

```javascript
kawo.tr( key )
```

```javascript
kawo.tr( error.displayError || image.getAttribute( 'failedMsg' ) )
```

```javascript
kawo.tr( test ? 'Something' : 'Something Else', '' )
```

```javascript
kawo.tr( `Top Posts Ranked by ${impressions}`, `根据${impressions}排名的帖子` )
```

```javascript
kawo.tr( 'Top Posts Ranked by ' + 'impressions', '根据' + '排名的帖子' )
```

```javascript
kawo.tr( 'Top Posts Ranked by ' + label.en, '根据' + label.cn + '排名的帖子' )
```

```javascript
kawo.tr( label.total.en, label.total.cn )
```