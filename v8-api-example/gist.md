# Array API
```
Local<Array> ret = Array::New(isolate, 2);
```

# Local API
```
static void GetPromiseDetails(const FunctionCallbackInfo<Value>& args) {
  Local<Promise> promise = args[0].As<Promise>();
```

# FunctionCallbackInfo
```
static void GetPromiseDetails(const FunctionCallbackInfo<Value>& args) {
  auto isolate = args.GetIsolate();
  Local<Context> context = args.GetIsolate()->GetCurrentContext();
  Local<Promise> promise = args[0].As<Promise>();
  ...
  Local<Array> ret = Array::New(isolate, 2);
  ret.Set(0, Integer::New(isolate, 0));
  ret.Set(1, Integer::New(isolate, 1));
  args.GetReturnValue().Set(ret);
}
```

# Interger API
```
Local<Number> one = Integer::New(isolate, 1);
```

# Bind method to object
```
inline void Environment::SetMethod(v8::Local<v8::Object> that,
                                   const char* name,
                                   v8::FunctionCallback callback) {
  v8::Local<v8::Function> function =
      NewFunctionTemplate(callback)->GetFunction();
  // kInternalized strings are created in the old space.
  const v8::NewStringType type = v8::NewStringType::kInternalized;
  v8::Local<v8::String> name_string =
      v8::String::NewFromUtf8(isolate(), name, type).ToLocalChecked();
  that->Set(name_string, function);
  function->SetName(name_string);  // NODE_SET_METHOD() compatibility.
}
```

# Bind method to class??
```
//that.prototype.name = callback
inline void Environment::SetProtoMethod(v8::Local<v8::FunctionTemplate> that,
                                        const char* name,
                                        v8::FunctionCallback callback) {
  v8::Local<v8::Signature> signature = v8::Signature::New(isolate(), that);
  v8::Local<v8::FunctionTemplate> t = NewFunctionTemplate(callback, signature);
  // kInternalized strings are created in the old space.
  const v8::NewStringType type = v8::NewStringType::kInternalized;
  v8::Local<v8::String> name_string =
      v8::String::NewFromUtf8(isolate(), name, type).ToLocalChecked();
  that->PrototypeTemplate()->Set(name_string, t);
  t->SetClassName(name_string);  // NODE_SET_PROTOTYPE_METHOD() compatibility.
}
```

# FunctionTemplate
Used to create functions at runtime and only one function will be created from a FunctionTemplate in a context. The lifetime of the created function is equal to the lifetime of the context.
```
  v8::Local<v8::FunctionTemplate> t = v8::FunctionTemplate::New(isolate);
```
A FunctionTemplate can have:
- Properties, will become the properties of created function
```
  t->Set(isolate, "func_property", v8::Number::New(isolate, 1));
```
- Instance Template, will be used to create object instances when the created function is used as a constructor.
```
  v8::Local<v8::ObjectTemplate> instance_t = t->InstanceTemplate();
  instance_t->SetAccessor(String::NewFromUtf8(isolate, "instance_accessor"),
                          InstanceAccessorCallback);
  instance_t->SetNamedPropertyHandler(PropertyHandlerCallback);
  instance_t->Set(String::NewFromUtf8(isolate, "instance_property"),
                  Number::New(isolate, 3));
```
- Prototype Template, will be used to create prototype object of the created function
```
  v8::Local<v8::Template> proto_t = t->PrototypeTemplate();
  proto_t->Set(isolate,
               "proto_method",
                v8::FunctionTemplate::New(isolate, InvokeCallback));
  proto_t->Set(isolate, "proto_const", v8::Number::New(isolate, 2));
```
Create a function and use it to create an object
```
  v8::Local<v8::Function> function = t->GetFunction();
  v8::Local<v8::Object> instance = function->NewInstance();
```

## inherit
```
 Local<FunctionTemplate> parent = t;
 Local<FunctionTemplate> child = FunctionTemplate::New();
 child->Inherit(parent);

 Local<Function> child_function = child->GetFunction();
 Local<Object> child_instance = child_function->NewInstance();
```

The following graph illustrates the semantics of inheritance
```
  FunctionTemplate Parent  -> Parent() . prototype -> { }
    ^                                                  ^
    | Inherit(Parent)                                  | .__proto__
    |                                                  |
  FunctionTemplate Child   -> Child()  . prototype -> { }
```

# set property to object
```
  //target is a Local<Object>
  target->DefineOwnProperty(
    context,
    OneByteString(env->isolate(), "pushValToArrayMax"),
    Integer::NewFromUnsigned(env->isolate(), NODE_PUSH_VAL_TO_ARRAY_MAX),
    v8::ReadOnly).FromJust();
```
