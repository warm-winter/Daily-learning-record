
第一次做d8的题

### 环境

参考了2019师傅的博客，只要梯子不拉垮，一早上足够了。

### 题目分析

petch中有这么一部分内容：

```
diff --git a/src/parsing/parser-base.h b/src/parsing/parser-base.h
index 3519599a88..f1ba0fb445 100644
--- a/src/parsing/parser-base.h
+++ b/src/parsing/parser-base.h
@@ -1907,10 +1907,8 @@ ParserBase<Impl>::ParsePrimaryExpression() {
       return ParseTemplateLiteral(impl()->NullExpression(), beg_pos, false);
 
     case Token::MOD:
-      if (flags().allow_natives_syntax() || extension_ != nullptr) {
-        return ParseV8Intrinsic();
-      }
-      break;
+      // Directly call %ArrayBufferDetach without `--allow-native-syntax` flag
+      return ParseV8Intrinsic();
 
     default:
       break;
diff --git a/src/parsing/parser.cc b/src/parsing/parser.cc
index 9577b37397..2206d250d7 100644
--- a/src/parsing/parser.cc
+++ b/src/parsing/parser.cc
@@ -357,6 +357,11 @@ Expression* Parser::NewV8Intrinsic(const AstRawString* name,
   const Runtime::Function* function =
       Runtime::FunctionForName(name->raw_data(), name->length());
 
+  // Only %ArrayBufferDetach allowed
+  if (function->function_id != Runtime::kArrayBufferDetach) {
+    return factory()->NewUndefinedLiteral(kNoSourcePosition);
+  }
+
   // Be more permissive when fuzzing. Intrinsics are not supported.
   if (FLAG_fuzzing) {
     return NewV8RuntimeFunctionForFuzzing(function, args, pos);
```
出题人把--allow-native-syntax支持删了，这样就把%DebugPrint和%SystemBreak砍掉了，没法调试。保留了%ArrayBufferDetach，并且不需要--allow-native-syntax参数。

为了方便调试，对patch文件进行一点修改，就是将上面这一部分删掉，这样就可以愉快的调试了。

接下来我们去看一下剩下的patch：
```
diff --git a/src/builtins/typed-array-set.tq b/src/builtins/typed-array-set.tq
++ b/src/builtins/typed-array-set.tq
    const utarget = %RawDownCast<AttachedJSTypedArray>(target);
      const utypedArray = %RawDownCast<AttachedJSTypedArray>(typedArray);