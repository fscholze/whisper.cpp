# whisper.objc

Minimal Obj-C application for automatic offline speech recognition.
The inference runs locally, on-device.

https://user-images.githubusercontent.com/1991296/197385372-962a6dea-bca1-4d50-bf96-1d8c27b98c81.mp4

Real-time transcription demo:

https://user-images.githubusercontent.com/1991296/204126266-ce4177c6-6eca-4bd9-bca8-0e46d9da2364.mp4

## Usage

This example uses the whisper.xcframework which needs to be built first using the following command:
```bash
./build-xcframework.sh
```

A model is also required to be downloaded and can be done using the following command:
```bash
./models/download-ggml-model.sh base.en
```

If you don't want to convert a Core ML model, you can skip this step by creating dummy model:
```bash
mkdir models/ggml-base.en-encoder.mlmodelc
```

### Core ML support
1. Follow all the steps in the `Usage` section, including adding the ggml model file.  
The ggml model file is required as the Core ML model is only used for the encoder. The
decoder which is in the ggml model is still required.
2. Follow the [`Core ML support` section of readme](../../README.md#core-ml-support) to convert the
model.
3. Add the Core ML model (`models/ggml-base.en-encoder.mlmodelc/`) to `whisper.swiftui.demo/Resources/models` **via Xcode**.

When the example starts running you should now see that it is using the Core ML model:
```console
whisper_init_state: loading Core ML model from '/Library/Developer/CoreSimulator/Devices/25E8C27D-0253-4281-AF17-C3F2A4D1D8F4/data/Containers/Bundle/Application/3ADA7D59-7B9C-43B4-A7E1-A87183FC546A/whisper.swiftui.app/models/ggml-base.en-encoder.mlmodelc'
whisper_init_state: first run on a device may take a while ...
whisper_init_state: Core ML model loaded
```

## Info za Serbski Model
1. Do `whisper.objc` rjadowaka recne modele `ggml-large-v3-turbo.bin` a `ggml-large-v3-turbo-encoder.mlmodelc` sunc
2. W `whisper.objc/ViewController` te mejno modela zmenic:
```
- (void)viewDidLoad {
    [super viewDidLoad];

    // whisper.cpp initialization
    {
        // load the model
        NSString *modelPath = [[NSBundle mainBundle] pathForResource:@"ggml-large-v3-turbo" ofType:@"bin"];

        // check if the model exists
        if (![[NSFileManager defaultManager] fileExistsAtPath:modelPath]) {
            NSLog(@"Model file not found");
            return;
        }
````
3. W `whisper.objc/ViewController` `params.translate` na `true` a `params.language` na `"czech"` zmenic:
```
- (IBAction)onTranscribe:(id)sender {
    if (stateInp.isTranscribing) {
        return;
    }

    NSLog(@"Processing %d samples", stateInp.n_samples);

    stateInp.isTranscribing = true;

    // dispatch the model to a background thread
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // process captured audio
        // convert I16 to F32
        for (int i = 0; i < self->stateInp.n_samples; i++) {
            self->stateInp.audioBufferF32[i] = (float)self->stateInp.audioBufferI16[i] / 32768.0f;
        }

        // run the model
        struct whisper_full_params params = whisper_full_default_params(WHISPER_SAMPLING_GREEDY);

        // get maximum number of threads on this device (max 8)
        const int max_threads = MIN(8, (int)[[NSProcessInfo processInfo] processorCount]);

        params.print_realtime   = true;
        params.print_progress   = false;
        params.print_timestamps = true;
        params.print_special    = false;
        params.translate        = true;
        params.language         = "czech";
        params.n_threads        = max_threads;
        params.offset_ms        = 0;
        params.no_context       = true;
        params.single_segment   = self->stateInp.isRealtime;
        params.no_timestamps    = params.single_segment;
```




