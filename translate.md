[Exposed=Window, SecureContext]
interface LanguageModel : EventTarget {
  static Promise<LanguageModel> create(optional LanguageModelCreateOptions options = {});
  static Promise<Availability> availability(optional LanguageModelCreateCoreOptions options = {});
  static Promise<LanguageModelParams?> params();

  // These will throw "NotSupportedError" DOMExceptions if role = "system"
  Promise<DOMString> prompt(
    LanguageModelPrompt input,
    optional LanguageModelPromptOptions options = {}
  );
  ReadableStream promptStreaming(
    LanguageModelPrompt input,
    optional LanguageModelPromptOptions options = {}
  );
  Promise<undefined> append(
    LanguageModelPrompt input,
    optional LanguageModelAppendOptions options = {}
  );

  Promise<double> measureInputUsage(
    LanguageModelPrompt input,
    optional LanguageModelPromptOptions options = {}
  );
  readonly attribute double inputUsage;
  readonly attribute unrestricted double inputQuota;
  attribute EventHandler onquotaoverflow;

  readonly attribute unsigned long topK;
  readonly attribute float temperature;

  Promise<LanguageModel> clone(optional LanguageModelCloneOptions options = {});
  undefined destroy();
};

[Exposed=Window, SecureContext]
interface LanguageModelParams {
  readonly attribute unsigned long defaultTopK;
  readonly attribute unsigned long maxTopK;
  readonly attribute float defaultTemperature;
  readonly attribute float maxTemperature;
};


callback LanguageModelToolFunction = Promise<DOMString> (any... arguments);

// A description of a tool call that a language model can invoke.
dictionary LanguageModelTool {
  required DOMString name;
  required DOMString description;
  // JSON schema for the input parameters.
  required object inputSchema;
  // The function to be invoked by user agent on behalf of language model.
  required LanguageModelToolFunction execute;
};

dictionary LanguageModelCreateCoreOptions {
  // Note: these two have custom out-of-range handling behavior, not in the IDL layer.
  // They are unrestricted double so as to allow +Infinity without failing.
  unrestricted double topK;
  unrestricted double temperature;

  sequence<LanguageModelExpected> expectedInputs;
  sequence<LanguageModelExpected> expectedOutputs;
  sequence<LanguageModelTool> tools;
};

dictionary LanguageModelCreateOptions : LanguageModelCreateCoreOptions {
  AbortSignal signal;
  CreateMonitorCallback monitor;

  sequence<LanguageModelMessage> initialPrompts;
};

dictionary LanguageModelPromptOptions {
  object responseConstraint;
  boolean omitResponseConstraintInput = false;
  AbortSignal signal;
};

dictionary LanguageModelAppendOptions {
  AbortSignal signal;
};

dictionary LanguageModelCloneOptions {
  AbortSignal signal;
};

dictionary LanguageModelExpected {
  required LanguageModelMessageType type;
  sequence<DOMString> languages;
};

// The argument to the prompt() method and others like it

typedef (
  sequence<LanguageModelMessage>
  // Shorthand for `[{ role: "user", content: [{ type: "text", value: providedValue }] }]`
  or DOMString
) LanguageModelPrompt;

dictionary LanguageModelMessage {
  required LanguageModelMessageRole role;

  // The DOMString branch is shorthand for `[{ type: "text", value: providedValue }]`
  required (DOMString or sequence<LanguageModelMessageContent>) content;

  boolean prefix = false;
};

dictionary LanguageModelMessageContent {
  required LanguageModelMessageType type;
  required LanguageModelMessageValue value;
};

enum LanguageModelMessageRole { "system", "user", "assistant" };

enum LanguageModelMessageType { "text", "image", "audio" };

typedef (
  ImageBitmapSource
  or AudioBuffer
  or BufferSource
  or DOMString
) LanguageModelMessageValue;
