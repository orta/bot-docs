# <a id="top"></a>Changes in dialogs: botbuilder-dotnet (4.6-preview)

Apparent changes:

- Dialog events and event bubbling
- Tags
- Bindings and skills
- Dialog containers
- Adaptive dialogs

Namespaces explored:

- [Microsoft.Bot.Builder.Dialogs](#ns-dialogs)
- [Microsoft.Bot.Builder.Dialogs.Adaptive](#ns-dialogs-adaptive)
- [Microsoft.Bot.Builder.Dialogs.Adaptive.Steps](#ns-dialogs-adaptive-steps)

## <a id="ns-dialogs"></a>[updated] Microsoft.Bot.Builder.Dialogs

### <a id="DialogManagerAdapter"></a>[new] class **DialogManagerAdapter** : BotAdapter

<details><summary>Public and protected members</summary>

```csharp
public DialogManagerAdapter() { }

public readonly List<Activity> Activities = new List<Activity>();

public override Task<ResourceResponse[]> SendActivitiesAsync(ITurnContext turnContext, Activity[] activities, CancellationToken cancellationToken) {…}

// Both of these throw a NotImplementedException.
public override Task<ResourceResponse> UpdateActivityAsync(ITurnContext turnContext, Activity activity, CancellationToken cancellationToken) {…}
public override Task DeleteActivityAsync(ITurnContext turnContext, ConversationReference reference, CancellationToken cancellationToken) {…}
```

</details>

- This is a transient, internal adapter used by the **[DialogManager](#DialogManager).RunAsync** method.
- On _send_ activities operations, ...assumes the activities have already gone wherever they're going to go, and just returns the **ResourceResponse** array containing their IDs.
- On _update_ and _delete_ activity operations, this throws.

back to [top](#top)

### <a id="IDialogDependencies"></a>public interface **IDialogDependencies**

<details><summary>Public and protected members</summary>

```csharp
List<IDialog> ListDependencies();
```

</details>

Implementing classes and interfaces:

- **Microsoft.Bot.Builder.Dialogs.[DialogCommand](#DialogCommand)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[EditSteps](#EditSteps)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[Foreach](#Foreach)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[ForeachPage](#ForeachPage)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[IfCondition](#IfCondition)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[SwitchCondition](#SwitchCondition)**

Not sure why stuff that derives from **DialogCommand** explicitly implements **IDialogDependencies**. Either only the concrete classes that need it should implement it, or just the abstract base class should implement it, but not both.

back to [top](#top)

### <a id="IDialog"></a>[updated] public interface **IDialog**

<details><summary>Public and protected members</summary>

```csharp
string Id { get; set; }
IBotTelemetryClient TelemetryClient { get; set; }
List<string> Tags { get; }
Dictionary<string, string> InputBindings { get; }
string OutputBinding { get; }

Task<DialogTurnResult> BeginDialogAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken));
Task<DialogTurnResult> ContinueDialogAsync(DialogContext dc, CancellationToken cancellationToken = default(CancellationToken));
Task<DialogTurnResult> ResumeDialogAsync(DialogContext dc, DialogReason reason, object result = null, CancellationToken cancellationToken = default(CancellationToken));
Task RepromptDialogAsync(ITurnContext turnContext, DialogInstance instance, CancellationToken cancellationToken = default(CancellationToken));
Task EndDialogAsync(ITurnContext turnContext, DialogInstance instance, DialogReason reason, CancellationToken cancellationToken = default(CancellationToken));

Task<bool> OnDialogEventAsync(DialogContext dc, DialogEvent e, CancellationToken cancellationToken);
```

</details>

back to [top](#top)

### <a id="Dialog"></a>[updated] public abstract class **Dialog** : [IDialog](#IDialog)

<details><summary>Public and protected members</summary>

```csharp
public static readonly DialogTurnResult EndOfTurn = new DialogTurnResult(DialogTurnStatus.Waiting);

private string id;
private IBotTelemetryClient _telemetryClient;

public Dialog(string dialogId = null) {…}

public string Id {get {…} set {…}}
public virtual IBotTelemetryClient TelemetryClient {get {…} set {…}}
public List<string> Tags { get; private set; } = new List<string>();
public Dictionary<string, string> InputBindings { get; set; } = new Dictionary<string, string>();
public string OutputBinding { get; set; }

public abstract Task<DialogTurnResult> BeginDialogAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken));
public virtual async Task<DialogTurnResult> ContinueDialogAsync(DialogContext dc, CancellationToken cancellationToken = default(CancellationToken)) {…}
public virtual async Task<DialogTurnResult> ResumeDialogAsync(DialogContext dc, DialogReason reason, object result = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
public virtual Task RepromptDialogAsync(ITurnContext turnContext, DialogInstance instance, CancellationToken cancellationToken = default(CancellationToken)) {…}
public virtual Task EndDialogAsync(ITurnContext turnContext, DialogInstance instance, DialogReason reason, CancellationToken cancellationToken = default(CancellationToken)) {…}

public virtual async Task<bool> OnDialogEventAsync(DialogContext dc, DialogEvent e, CancellationToken cancellationToken) {…}
protected virtual Task<bool> OnPreBubbleEvent(DialogContext dc, DialogEvent e, CancellationToken cancellationToken) {…}
protected virtual Task<bool> OnPostBubbleEvent(DialogContext dc, DialogEvent e, CancellationToken cancellationToken) {…}

protected virtual string OnComputeId() {…}
protected virtual string BindingPath() {…}

protected void RegisterSourceLocation(string path, int lineNumber) {…}
```

</details>

back to [top](#top)

### <a id="DialogCommand"></a>[new] public abstract class **DialogCommand** : [Dialog](#Dialog), [IDialogDependencies](#IDialogDependencies)

<details><summary>Public and protected members</summary>

```csharp
public override Task<DialogTurnResult> BeginDialogAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}

public virtual List<IDialog> ListDependencies() {…}

protected abstract Task<DialogTurnResult> OnRunCommandAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken));

protected async Task<DialogTurnResult> EndParentDialogAsync(DialogContext dc, object result = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
protected async Task<DialogTurnResult> ReplaceParentDialogAsync(DialogContext dc, string dialogId, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
protected async Task<DialogTurnResult> RepeatParentDialogAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
protected async Task<DialogTurnResult> CancelAllParentDialogsAsync(DialogContext dc, object result = null, string eventName = "cancelDialog", object eventValue = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
```

</details>

This appears to be a base class for "command-like" dialogs. Reading between the lines, these are steps that are available to the Bot Composer.

Derived classes:

- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[BaseInvokeDialog](#BaseInvokeDialog)**
  - **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[BeginDialog](#BeginDialog)**
  - **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[ReplaceDialog](#ReplaceDialog)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[CancelAllDialogs](#CancelAllDialogs)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[CodeStep](#CodeStep)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[DebugBreak](#DebugBreak)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[DeleteProperty](#DeleteProperty)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[EditArray](#EditArray)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[EditSteps](#EditSteps)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[EmitEvent](#EmitEvent)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[EndDialog](#EndDialog)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[EndTurn](#EndTurn)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[Foreach](#Foreach)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[ForeachPage](#ForeachPage)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[HttpRequest](#HttpRequest)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[IfCondition](#IfCondition)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[InitProperty](#InitProperty)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[LogStep](#LogStep)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[RepeatDialog](#RepeatDialog)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[SendActivity](#SendActivity)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[SetProperty](#SetProperty)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[SwitchCondition](#SwitchCondition)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[TraceActivity](#TraceActivity)**

back to [top](#top)

### <a id="DialogContainer"></a>[new] public abstract class **DialogContainer** : [Dialog](#Dialog)

<details><summary>Public and protected members</summary>

```csharp
protected readonly DialogSet _dialogs = new DialogSet();

public DialogContainer(string dialogId = null) : base(dialogId) {…}

public abstract DialogContext CreateChildContext(DialogContext dc);

public virtual Dialog AddDialog(IDialog dialog) {…}
public IDialog FindDialog(string dialogId) {…}
```

</details>

**DialogContainer** is now a common base class for [component](#ComponentDialog) and [adaptive](#AdaptiveDialog) dialogs.

- It defines an inner dialog set.
- I'm not sure why it doesn't override the **[Dialog](#Dialog).TelemetryClient** setter to apply the telemetry client to its child dialogs. The implementations in **ComponentDialog** and **AdaptiveDialog**, are slightly differently but logically identical.

back to [top](#top)

### <a id="ComponentDialog"></a>[updated] public class **ComponentDialog** : [DialogContainer](#DialogContainer)

<details><summary>Public and protected members</summary>

```csharp
public const string PersistedDialogState = "dialogs";

public ComponentDialog(string dialogId = null) : base(dialogId) {…}

public string InitialDialogId { get; set; }

public new IBotTelemetryClient TelemetryClient { get {…} set {…} }

public override async Task<DialogTurnResult> BeginDialogAsync(DialogContext outerDc, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
public override async Task<DialogTurnResult> ContinueDialogAsync(DialogContext outerDc, CancellationToken cancellationToken = default(CancellationToken)) {…}
public override async Task<DialogTurnResult> ResumeDialogAsync(DialogContext outerDc, DialogReason reason, object result = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
public override async Task RepromptDialogAsync(ITurnContext turnContext, DialogInstance instance, CancellationToken cancellationToken = default(CancellationToken)) {…}
public override async Task EndDialogAsync(ITurnContext turnContext, DialogInstance instance, DialogReason reason, CancellationToken cancellationToken = default(CancellationToken)) {…}

protected async Task EnsureInitialized(DialogContext outerDc) {…}

public override Dialog AddDialog(IDialog dialog) {…}
public IDialog FindDialog(string dialogId) {…}

public override DialogContext CreateChildContext(DialogContext dc) {…}

protected virtual Task OnInitialize(DialogContext dc) {…}

protected virtual Task<DialogTurnResult> OnBeginDialogAsync(DialogContext innerDc, object options, CancellationToken cancellationToken = default(CancellationToken)) {…}
protected virtual Task<DialogTurnResult> OnContinueDialogAsync(DialogContext innerDc, CancellationToken cancellationToken = default(CancellationToken)) {…}
protected virtual Task OnEndDialogAsync(ITurnContext context, DialogInstance instance, DialogReason reason, CancellationToken cancellationToken = default(CancellationToken)) {…}
protected virtual Task OnRepromptDialogAsync(ITurnContext turnContext, DialogInstance instance, CancellationToken cancellationToken = default(CancellationToken)) {…}

protected virtual Task<DialogTurnResult> EndComponentAsync(DialogContext outerDc, object result, CancellationToken cancellationToken) {…}

protected override string OnComputeId() {…}
```

</details>

back to [top](#top)

### <a id="DialogContext"></a>[updated] public class **DialogContext**

<details><summary>Public and protected members</summary>

```csharp
private List<string> activeTags = new List<string>();

public DialogContext(DialogSet dialogs, DialogContext parentDialogContext, DialogState state, IDictionary<string, object> conversationState = null, IDictionary<string, object> userState = null, IDictionary<string, object> settings = null) {…}
public DialogContext(DialogSet dialogs, ITurnContext turnContext, DialogState state, IDictionary<string, object> conversationState = null, IDictionary<string, object> userState = null, IDictionary<string, object> settings = null) {…}

public DialogContext Parent { get; set; }
public DialogSet Dialogs { get; private set; }
public ITurnContext Context { get; private set; }
public IList<DialogInstance> Stack { get; private set; }
public DialogContextState State { get; private set; }
public DialogContext Child { get {…} }
public DialogInstance ActiveDialog { get {…} }
public List<string> ActiveTags { get {…} }
public Dictionary<string, object> DialogState { get {…} }

public async Task<DialogTurnResult> BeginDialogAsync(string dialogId, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
public async Task<DialogTurnResult> PromptAsync(string dialogId, PromptOptions options, CancellationToken cancellationToken = default(CancellationToken)) {…}
public async Task<DialogTurnResult> ContinueDialogAsync(CancellationToken cancellationToken = default(CancellationToken)) {…}
public async Task<DialogTurnResult> EndDialogAsync(object result = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
public async Task<DialogTurnResult> CancelAllDialogsAsync(string eventName = DialogEvents.CancelDialog, object eventValue = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
public async Task<DialogTurnResult> ReplaceDialogAsync(string dialogId, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
public async Task RepromptDialogAsync(CancellationToken cancellationToken = default(CancellationToken)) {…}

public IDialog FindDialog(string dialogId) {…}
public async Task<bool> EmitEventAsync(string name, object value = null, bool bubble = true, bool fromLeaf = false, CancellationToken cancellationToken = default(CancellationToken)) {…}
protected virtual bool ShouldInheritState(IDialog dialog) {…}
private async Task EndActiveDialogAsync(DialogReason reason, object result = null, CancellationToken cancellationToken = default(CancellationToken)) {…}

public class DialogEvents
{
    public const string BeginDialog = "beginDialog";
    public const string ResumeDialog = "resumeDialog";
    public const string RepromptDialog = "repromptDialog";
    public const string CancelDialog = "cancelDialog";
    public const string EndDialog = "endDialog";
    public const string ActivityReceived = "activityReceived";
}
```

</details>

back to [top](#top)

### <a id="DialogState"></a>[updated] public class **DialogState**

<details><summary>Public and protected members</summary>

```csharp
public DialogState() : this(null) { }
public DialogState(IList<DialogInstance> stack) {…}

public IList<DialogInstance> DialogStack { get; set; } = new List<DialogInstance>();
public IDictionary<string, object> ConversationState { get; set; } = new Dictionary<string, object>();
public IDictionary<string, object> UserState { get; set; } = new Dictionary<string, object>();
```

</details>

- The additon of the **ConversationState** and **UserState** properties looks ugly. I'll try to get them to change the names so they don't actively collide with the state management classes of the same name.

### <a id="StoredBotState"></a>[new] public class **StoredBotState**

<details><summary>Public and protected members</summary>

```csharp
public IDictionary<string, object> UserState { get; set; }
public IDictionary<string, object> ConversationState { get; set; }
public IList<DialogInstance> DialogStack { get; set; }
```

</details>

back to [top](#top)

### <a id="DialogEvent"></a>[new] public class **DialogEvent**

<details><summary>Public and protected members</summary>

```csharp
public bool Bubble { get; set; }  // Whether to propagate events to parent contexts.
public string Name { get; set; }  // Event name.
public object Value { get; set; } // Optional. Event value.
```

</details>

### <a id="DialogManager"></a>[new] public class **DialogManager**

<details><summary>Public and protected members</summary>

```csharp
public DialogManager(IDialog rootDialog = null) {…}

public IDialog RootDialog { get {…} set {…} }

public async Task<DialogManagerResult> RunAsync(Activity activity, StoredBotState state = null) {…}
public async Task<DialogManagerResult> OnTurnAsync(ITurnContext context, StoredBotState storedState = null, CancellationToken cancellationToken = default(CancellationToken)) {…}

private static async Task<StoredBotState> LoadBotState(IStorage storage, BotStateStorageKeys keys) {…}
private static async Task SaveBotState(IStorage storage, StoredBotState newState, BotStateStorageKeys keys) {…}

private static BotStateStorageKeys ComputeKeys(ITurnContext context) {…}
```

</details>

This appears to take over from the **DialogExtensions.RunAsync** extension method, and maybe the **IBot.OnTurnAsync** and **BotFrameworkAdapter.ProcessActivityAsync** methods, too.

- This re-introduces the concept of a _root dialog_ that was a feature of the v3 library.
- Use **RunAsync** to "send" and activity to [the adapter via] the dialog manager.
  - This takes an **Activity** and a [**StoredBotState**](#StoredBotState) as input.
    - If **StoredBotState** is null, the new state model is populated via the **TurnContext** and a **saveState** flag is set.
    - State management objects appear to be thrown out, and the storage layer is accessed directly from the turn context.
  - This creates internal and transient [**DialogManagerAdapter**](#DialogManagerAdapter) and **TurnContext** objects.
    - I'm not sure why. As an end run around middleware an the "old" state implementation?
    - There are also a bunch of protocol-level features that are not supported, such as continue conversation, and so on.
- **OnTurnAsync** "processes" the activity.
  - It creates a **DialogContext** for the turn.
  - It "emits" an **ActivityReceived** dialog event.
  - It continues the active dialog; else it starts the "root dialog".
  - If the **saveState** flag is set, saves state to the turn context.
  - It returns a [**DialogManagerResult**](#DialogManagerResult) object.
- This effectively accepts and propagates two different versions of state.

back to [top](#top)

### <a id="DialogManagerResult"></a>[new] public class **DialogManagerResult**

<details><summary>Public and protected members</summary>

```csharp
public DialogTurnResult TurnResult { get; set; }
public Activity[] Activities { get; set; }
public StoredBotState NewState { get; set; }
```

</details>

back to [top](#top)

### <a id="DialogTurnResult"></a>[updated] public class **DialogTurnResult**

<details><summary>Public and protected members</summary>

```csharp
public DialogTurnResult(DialogTurnStatus status, object result = null) {…}

public DialogTurnStatus Status { get; set; }
public object Result { get; set; }
public bool ParentEnded { get; set; }
```

</details>

back to [top](#top)

## <a id="ns-dialogs-adaptive"></a>[new] Microsoft.Bot.Builder.Dialogs.Adaptive

### <a id="AdaptiveDialog"></a>public class **AdaptiveDialog** : [DialogContainer](#DialogContainer)

<details><summary>Public and protected members</summary>

```csharp
public IStatePropertyAccessor<BotState> BotState { get; set; }
public IStatePropertyAccessor<Dictionary<string, object>> UserState { get; set; }

public IRecognizer Recognizer { get; set; }
public ILanguageGenerator Generator { get; set; }

public List<IDialog> Steps { get; set; } = new List<IDialog>();
public virtual List<IRule> Rules { get; set; } = new List<IRule>();
public bool AutoEndDialog { get; set; } = true;
public IRuleSelector Selector { get; set; }

public string DefaultResultProperty { get; set; } = "dialog.result";

public override IBotTelemetryClient TelemetryClient { get {…} set {…} }

public AdaptiveDialog(string dialogId = null, [CallerFilePath] string callerPath = "", [CallerLineNumber] int callerLine = 0) : base(dialogId) {…}

public override async Task<DialogTurnResult> BeginDialogAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
public override async Task<DialogTurnResult> ContinueDialogAsync(DialogContext dc, CancellationToken cancellationToken = default(CancellationToken)) {…}
public override async Task<DialogTurnResult> ResumeDialogAsync(DialogContext dc, DialogReason reason, object result = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
public override async Task RepromptDialogAsync(ITurnContext turnContext, DialogInstance instance, CancellationToken cancellationToken = default(CancellationToken)) {…}

protected override async Task<bool> OnPreBubbleEvent(DialogContext dc, DialogEvent dialogEvent, CancellationToken cancellationToken = default(CancellationToken)) {…}
protected override async Task<bool> OnPostBubbleEvent(DialogContext dc, DialogEvent dialogEvent, CancellationToken cancellationToken = default(CancellationToken)) {…}

protected async Task<bool> ProcessEventAsync(SequenceContext sequenceContext, DialogEvent dialogEvent, bool preBubble, CancellationToken cancellationToken = default(CancellationToken)) {…}

public void AddRule(IRule rule) {…}
public void AddRules(IEnumerable<IRule> rules) {…}

public void AddDialogs(IEnumerable<IDialog> dialogs) {…}

protected override string OnComputeId() {…}
private string GetUniqueInstanceId(DialogContext dc) {…}

public override DialogContext CreateChildContext(DialogContext dc) {…}

protected async Task<DialogTurnResult> ContinueStepsAsync(DialogContext dc, object options, CancellationToken cancellationToken) {…}
protected async Task<bool> EndCurrentStepAsync(SequenceContext sequenceContext, CancellationToken cancellationToken = default(CancellationToken)) {…}
protected async Task<DialogTurnResult> OnEndOfStepsAsync(SequenceContext sequenceContext, CancellationToken cancellationToken = default(CancellationToken)) {…}
protected async Task<RecognizerResult> OnRecognize(SequenceContext sequenceContext, CancellationToken cancellationToken = default(CancellationToken)) {…}
```

</details>

back to [top](#top)

### <a id="SequenceContext"></a>public class **SequenceContext** : [DialogContext](#DialogContext)

<details><summary>Public and protected members</summary>

```csharp
public AdaptiveDialogState Plans { get; private set; }

public List<StepState> Steps { get; set; }

public List<StepChangeList> Changes { get {…} private set {…} }

public SequenceContext(DialogSet dialogs, DialogContext dc, DialogState state, List<StepState> steps, string changeKey, DialogSet stepDialogs)
    : base(dialogs, dc.Context, state, conversationState: dc.State.Conversation, userState: dc.State.User, settings: dc.State.Settings) {…}

public void QueueChanges(StepChangeList changes) {…}

public async Task<bool> ApplyChangesAsync(CancellationToken cancellationToken = default(CancellationToken)) {…}

public SequenceContext InsertSteps(List<StepState> steps) {…}
public SequenceContext InsertStepsBeforeTags(List<string> tags, List<StepState> steps) {…}
public SequenceContext AppendSteps(List<StepState> steps) {…}

public SequenceContext EndSequence(List<StepState> steps) {…}
public SequenceContext ReplaceSequence(List<StepState> steps) {…}

protected override bool ShouldInheritState(IDialog dialog) {…}
```

</details>

back to [top](#top)

### <a id="AdaptiveEvents"></a>public class **AdaptiveEvents** : [DialogContext](#DialogContext).DialogEvents

<details><summary>Public and protected members</summary>

```csharp
public const string RecognizedIntent = "recognizedIntent";
public const string UnknownIntent = "unknownIntent";
public const string SequenceStarted = "stepsStarted";
public const string SequenceEnded = "stepsEnded";
```

</details>

back to [top](#top)

### <a id="StepState"></a>public class **StepState** : [DialogState](#DialogState)

<details><summary>Public and protected members</summary>

```csharp
public AdaptiveDialogState() { }

public dynamic Options { get; set; }
public List<StepState> Steps { get; set; } = new List<StepState>();
public object Result { get; set; }
```

</details>

- Not sure if the use of `dynamic` here is a bug. The SDK has actively avoided the use of `dynamic` up to this point.

back to [top](#top)

### <a id="StepChangeTypes"></a>public enum **StepChangeTypes**

<details><summary>Public and protected members</summary>

```csharp
public enum StepChangeTypes
{
    InsertSteps,
    InsertStepsBeforeTags,
    AppendSteps,
    EndSequence,
    ReplaceSequence,
}
```

</details>

back to [top](#top)

### <a id="StepChangeList"></a>public class **StepChangeList**

<details><summary>Public and protected members</summary>

```csharp
public StepChangeTypes ChangeType { get; set; } = StepChangeTypes.InsertSteps;
public List<StepState> Steps { get; set; } = new List<StepState>();
public List<string> Tags { get; set; } = new List<string>();
public Dictionary<string, object> Turn { get; set; }
```

</details>

back to [top](#top)

## <a id="ns-dialogs-adaptive-steps"></a>[new] Microsoft.Bot.Builder.Dialogs.Adaptive.Steps

### <a id="BaseInvokeDialog"></a>public abstract class **BaseInvokeDialog** : [DialogCommand](#DialogCommand)

<details><summary>Public and protected members</summary>

```csharp
protected string dialogIdToCall;

public object Options { get; set; }

public IDialog Dialog { get; set; }
public string Property { get {…} set {…} }

public BaseInvokeDialog(string dialogIdToCall = null, string property = null, object options = null) : base() {…}

public override List<IDialog> ListDependencies() {…}

protected override string OnComputeId() {…}

protected IDialog ResolveDialog(DialogContext dc) {…}

protected void BindOptions(DialogContext dc) {…}
```

</details>

- Represents a step that calls/invokes another dialog.
- The **Property** property sets up a binding between the parent and child contexts. It is used to set the initial/input value in the child, and the final/output value in the parent.

Derived classes:

- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[BeginDialog](#BeginDialog)**
- **Microsoft.Bot.Builder.Dialogs.Adaptive.Steps.[ReplaceDialog](#ReplaceDialog)**

back to [top](#top)

### <a id="BeginDialog"></a>public class **BeginDialog** : [BaseInvokeDialog](#BaseInvokeDialog)

<details><summary>Public and protected members</summary>

```csharp
public BeginDialog(string dialogIdToCall = null, string property = null, object options = null, [CallerFilePath] string callerPath = "", [CallerLineNumber] int callerLine = 0)
    : base(dialogIdToCall, property, options) {…}

protected async override Task<DialogTurnResult> OnRunCommandAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
```

</details>

back to [top](#top)

### <a id="CancelAllDialogs"></a>public class **CancelAllDialogs** : [DialogCommand](#DialogCommand)

<details><summary>Public and protected members</summary>

```csharp
public CancelAllDialogs([CallerFilePath] string callerPath = "", [CallerLineNumber] int callerLine = 0) : base() {…}

public string EventName { get; set; }
public string EventValue { get; set; }

protected override async Task<DialogTurnResult> OnRunCommandAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}

protected override string OnComputeId() {…}
```

</details>

- When _run_, calls **[DialogCommand](#DialogCommand).CancelAllParentDialogsAsync** with `eventName: EventName ?? "cancelDialog"` and `eventValue: EventValue`.

back to [top](#top)

### <a id="CodeStep"></a>public class **CodeStep** : [DialogCommand](#DialogCommand)

<details><summary>Public and protected members</summary>

```csharp
public CodeStep(CodeStepHandler codeHandler, [CallerFilePath] string callerPath = "", [CallerLineNumber] int callerLine = 0) : base() {…}

protected override async Task<DialogTurnResult> OnRunCommandAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}

protected override string OnComputeId() {…}
```

</details>

- When _run_, runs a _code handler_.

    ```csharp
    using CodeStepHandler = Func<DialogContext, object, Task<DialogTurnResult>>;
    ```

back to [top](#top)

### <a id="DebugBreak"></a>public class **DebugBreak** : [DialogCommand](#DialogCommand)

<details><summary>Public and protected members</summary>

```csharp
public DebugBreak([CallerFilePath] string callerPath = "", [CallerLineNumber] int callerLine = 0) {…}

protected override async Task<DialogTurnResult> OnRunCommandAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
```

</details>

- When _run_ and there's an attached **System.Diagnostics.Debugger**, calls **Debugger.Break**.

back to [top](#top)

### <a id="DeleteProperty"></a>public class **DeleteProperty** : [DialogCommand](#DialogCommand)

<details><summary>Public and protected members</summary>

```csharp
public string Property { get; set; }

public DeleteProperty([CallerFilePath] string callerPath = "", [CallerLineNumber] int callerLine = 0) : base() {…}
public DeleteProperty(string property, [CallerFilePath] string callerPath = "", [CallerLineNumber] int callerLine = 0) : base() {…}

protected override async Task<DialogTurnResult> OnRunCommandAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
```

</details>

- When _run_ and our dialog context is a [**SequenceContext**](#SequenceContext), deletes a property from DialogManager-style state.

back to [top](#top)

### <a id="EditArray"></a>public class **EditArray** : [DialogCommand](#DialogCommand)

<details><summary>Public and protected members</summary>

```csharp
public enum ArrayChangeType { Push, Pop, Take, Remove, Clear }

public EditArray([CallerFilePath] string callerPath = "", [CallerLineNumber] int callerLine = 0) : base() {…}

protected override string OnComputeId() {…}

public ArrayChangeType ChangeType { get; set; }
public string ArrayProperty { get {…} set {…} }
public string ResultProperty { get {…} set {…} }
public string Value { get {…} set {…} }

public EditArray(ArrayChangeType changeType, string arrayProperty = null, string value = null, string resultProperty = null) : base() {…}

protected override async Task<DialogTurnResult> OnRunCommandAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}
```

</details>

The _array_ [property] we're working with is a **Newtonsoft.Json.Linq.JArray**.

- Pop removes and returns a value from the end.
- Push adds a value to the end.
- Take removes and returns a value from the start.
- Remove finds the first instance of a value in the array and removes it. Returns true if an item was removed.
- Clear empties the array. Returns true if any items were removed.

Push and pop describe a stack, and push and take describe a queue. Not sure why more array operations are not implemented, but they are easy to add.

back to [top](#top)

### <a id="EditSteps"></a>public class **EditSteps** : [DialogCommand](#DialogCommand), [IDialogDependencies](#IDialogDependencies)

<details><summary>Public and protected members</summary>

```csharp
public List<IDialog> Steps { get; set; } = new List<IDialog>();

public StepChangeTypes ChangeType { get; set; }

public EditSteps([CallerFilePath] string sourceFilePath = "", [CallerLineNumber] int sourceLineNumber = 0) : base() {…}

protected override async Task<DialogTurnResult> OnRunCommandAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken)) {…}

protected override string OnComputeId() {…}

public override List<IDialog> ListDependencies() {…}
```

</details>

- When _run_ and our dialog context is a [**SequenceContext**](#SequenceContext), queues the changes [on the sequence context] and then ends itself.
- Not sure when these queued changes get applied.

back to [top](#top)