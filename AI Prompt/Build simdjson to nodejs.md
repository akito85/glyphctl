How to implement simdjson, and simdutf c++ source code into nodejs application from
scratch without simdjson npm package? the binding.cc should use recent simdjson
and simputf functions, and recent c++ code writing.

using binding.gyp modern c++, target_name simdjsonaddon, compiled with clang (llvm binary),
support multithreading and multi architecture.

using binding.cc with the following functions:
    parse JSON from api http response
    parse JSON from database
    parse JSON from websocket
    parse JSON from file
    parse JSON from buffer

optimized for SIMD acceleration with dynamic selection
optimized for batch processing efficiency
optimized for multithreading
optimized for streaming JSON parsing for large files
optimized for memory safety, pervents buff overflows, and overhead with zero-copy parsing
optimized for direct memory conversion, no unecessary file i/o
optimized for Nodejs worker threads to offload json parsing to separate threads
optimized for chunk-based parallel JSON parsing using simdjson

avoid error below:
error string_to_json
error doc when parsing fails and malformed json inputs
conversion from ‘simdjson::simdjson_result<simdjson::haswell::ondemand::document>’ to non-scalar type ‘simdjson::haswell::ondemand::document’ requested simdjson::ondemand::document doc = parser.iterate(json_str);
in expansion of macro ‘NAPI_THROW_IF_FAILED’ NAPI_THROW_IF_FAILED(*this, status, Object());
In member function ‘virtual void Napi::AsyncWorker::OnExecute(Napi::Env)’: ‘e’ was not declared in this scope SetError(e.what());
exception handling disabled, use ‘-fexceptions’ to enable if ((status) != napi_ok) throw Napi::Error::New(env);
use of deleted function ‘simdjson::fallback::ondemand::document::document(const simdjson::fallback::ondemand::document&)’ auto doc = doc_result.value(); 
simdjson_inline document(const document &other) noexcept = delete; // pass your documents by reference, not by copy
use ‘-fdiagnostics-all-candidates’ to display considered candidates auto doc = doc_result.value(); // FIX: Get the document properly
use of deleted function ‘simdjson::fallback::ondemand::document::document(const simdjson::fallback::ondemand::document&)’auto doc = doc_result.value(); // FIX: Use .value()
invalid initialization of reference of type ‘const std::string&’ {aka ‘const std::__cxx11::basic_string<char>&’} from expression of type ‘simdjson::padded_string’ std::string parsed = parseJson(json_data);
no matching function for call to ‘simdjson::padded_string::load(const char*&, std::size_t&)’ simdjson::padded_string json_data = simdjson::padded_string::load(input, length);
candidate: ‘static simdjson::simdjson_result<simdjson::padded_string> simdjson::padded_string::load(std::string_view)’ inline simdjson_result<padded_string> padded_string::load(std::string_view filename) noexcept
'copy’ is not a member of ‘simdjson::padded_string’ simdjson::padded_string json_data = simdjson::padded_string::copy(input, length);  // Copy input buffer
‘struct simdjson::simdjson_result<simdjson::padded_string>’ has no member named ‘data’ return Napi::String::New(env, parseJson(json_data.data(), json_data.size()));
‘copy’ is not a member of ‘simdjson::padded_string’ simdjson::padded_string json_data = simdjson::padded_string::copy(jsonInputs[i]);
‘make_owned’ is not a member of ‘simdjson::padded_string’ auto json_data = simdjson::padded_string::make_owned(input, length);  // Safe ownership transfer
comparison of integer expressions of different signedness: ‘int’ and ‘std::size_t’ {aka ‘long unsigned int’} [-Wsign-compare] if (numThreads > numInputs) numThreads = numInputs; // Avoid over-threading
cannot convert ‘Napi::CallbackInfo’ to ‘const char*’ in argument passing using ReturnType = decltype(cb(CallbackInfo(nullptr, nullptr)));
type ‘<type error>’ argument given to ‘delete’, expected pointer delete callbackData;
required from here std::string output = std::string(doc);
static assertion failed: The get method with given type is not implemented by the simdjson library. The supported types are ondemand::object, ondemand::array, raw_json_string, std::string_view, uint64_t, int64_t, double, and bool. We recommend you use get_double(), get_bool(), get_uint64(), get_int64(),  get_object(), get_array(), get_raw_json_string(), or get_string() instead of the get template. You may also add support for custom types, see our documentation.
static_assert(!sizeof(T), "The get method with given type is not implemented by the simdjson library. "
simdjson_warn_unused simdjson_result<document> iterate(padded_string &&json) & noexcept = delete;
basic_string<char>::basic_string(simdjson::simdjson_result<std::basic_string_view<char> >)’ return Napi::String::New(env, std::string(doc.raw_json()));
‘struct simdjson::simdjson_result<simdjson::fallback::ondemand::document>’ has no member named ‘raw_json’ return std::string(result.raw_json().value_unsafe());
‘std::string ensure_utf8(const std::string&)’: call of overloaded ‘validate_utf8(const char*, std::__cxx11::basic_string<char>::size_type)’ is ambiguous
ignoring return value of ‘std::size_t simdutf::convert_utf16be_to_utf8(const char16_t*, std::size_t, char*)’ declared with attribute ‘warn_unused_result’ [-Wunused-result]
error: ‘count_utf16le_to_utf8’ is not a member of ‘simdutf’; did you mean ‘convert_utf16le_to_utf8’?
simdjson::simdjson_result<simdjson::haswell::ondemand::document>::simdjson_result(const simdjson::simdjson_result<simdjson::haswell::ondemand::document>&)’ is implicitly deleted because the default definition would be ill-formed
../binding.cc: In function ‘Napi::Value ParseJsonString(const Napi::CallbackInfo&)’:  
../binding.cc:30:45: error: ‘copy’ is not a member of ‘simdjson::padded_string’  
30 |         padded_string json = padded_string::copy(jsonStr);  // **Zero-copy parsing**
../binding.cc: In function ‘Napi::Value ParseJsonString(const Napi::CallbackInfo&)’:  
../binding.cc:30:45: error: ‘make_padded’ is not a member of ‘simdjson::padded_string’  
30 |         padded_string json = padded_string::make_padded(jsonStr); // **Zero copy parsing**c




Binding.cc sanity check 
1️⃣ 🔄 Thread-local parser 
✅ Avoids frequent memory allocations 
✅ Maximizes performance

2️⃣ 🛠 std::optional<> for safer error handling 
✅ Prevents unnecessary string copies 
✅ Avoids segmentation faults 

3️⃣ 📂 Efficient File Handling 
✅ Reads JSON files in binary mode 
✅ Ensures correct memory allocation 

4️⃣ ⚡ Async & Sync in One Function
✅ No need for separate functions
✅ Works for JSON strings, streams, files 
✅ Directly Returns JavaScript Objects (No need for JSON.parse()) 
✅ No Unnecessary String Conversions (Avoids serialization overhead)
✅ Thread-local simdjson::ondemand::parser (Avoids frequent memory allocations)
✅ Handles JSON Strings, Files, and Streams in One Function
✅ Preserves Maximum Performance with simdjson & simdutf APIs 
✅ Memory Safety: Ensured no invalid memory references. 
✅ Garbage Collection Safety: Returning a safe JS object. 
✅ Thread-Safe Parser Use: Ensured parser reuse is correct. 
✅ Benchmark Fix: Prevented crashing when calling JSON.parse() dont repeat previous error build. use latest simdjson and simpdutf api (2024) - 
✅ 
- ✅ **Ensures Memory Safety for `padded_string`.**
- ✅ **Supports both Sync & Async calls.**
- ✅ **No dangling references or invalid memory access.**
- ✅ **Directly returns JavaScript objects (no need for `JSON.parse()`).**
- ✅ **Benchmark stability improvements.**
