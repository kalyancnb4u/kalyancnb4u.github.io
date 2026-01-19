---
title: "Complete Go Mastery Part 14: Machine Learning & AI Integration"
date: 2025-01-13 18:00:00 +0530
categories: [Programming, Go, Machine-Learning]
tags: [golang, go, machine-learning, ai, neural-networks, tensorflow, nlp, computer-vision]
---

# Part 14: Machine Learning & AI Integration

## Overview

This part explores machine learning and AI with Go, covering traditional ML algorithms, neural networks, model serving, and integration with popular ML frameworks. You'll learn when to use Go for ML, how to build and deploy models, and how to integrate Go applications with AI services.

**What You'll Learn:**
- ML fundamentals for Go developers
- Traditional ML with GoLearn (classification, regression, clustering)
- Neural networks with Gorgonia
- TensorFlow and ONNX Runtime integration
- Natural Language Processing (NLP)
- Computer Vision with gocv
- Model serving and deployment
- MLOps patterns in Go
- AI API integration (OpenAI, Anthropic)

**Prerequisites:**
- Part 2 (Concurrency)
- Part 5 (Architecture patterns)
- Basic understanding of ML concepts
- Linear algebra basics (helpful but not required)

---

## 14.1 Machine Learning Fundamentals

### ML Concepts for Go Developers

**Machine Learning Types:**
- **Supervised Learning**: Labeled training data (classification, regression)
- **Unsupervised Learning**: Unlabeled data (clustering, dimensionality reduction)
- **Reinforcement Learning**: Learn from rewards/penalties

**The ML Workflow:**
```
Data Collection → Preprocessing → Training → Evaluation → Deployment → Monitoring
```

### Go's Role in ML

**Go is Good For:**
- Model serving/inference (production deployment)
- Data preprocessing pipelines
- ML infrastructure (Kubernetes, distributed systems)
- Real-time prediction APIs
- ETL for ML data

**Go is Less Suited For:**
- Model training (Python ecosystem is richer)
- Research and experimentation
- Deep learning training (GPU support limited)

**Common Pattern:**
- Train models in Python (TensorFlow, PyTorch)
- Serve models in Go (fast, concurrent, easy deployment)

### ML Libraries in Go

**Traditional ML:**
- GoLearn - Classification, regression, clustering
- Gonum - Numerical computing, linear algebra
- Gota - DataFrames (like pandas)

**Deep Learning:**
- Gorgonia - Neural networks, automatic differentiation
- TensorFlow Go bindings
- ONNX Runtime Go

**Computer Vision:**
- gocv - OpenCV bindings

**NLP:**
- prose - Text processing
- go-nlp - Natural language processing

---

## 14.2 Traditional ML with GoLearn

### Installation

```bash
go get -u github.com/sjwhitworth/golearn/...
```

### Data Loading and Preparation

```go
package main

import (
    "fmt"
    
    "github.com/sjwhitworth/golearn/base"
    "github.com/sjwhitworth/golearn/evaluation"
    "github.com/sjwhitworth/golearn/knn"
)

func main() {
    // Load data from CSV
    rawData, err := base.ParseCSVToInstances("iris.csv", true)
    if err != nil {
        panic(err)
    }
    
    fmt.Println(rawData)
    
    // Split into training and test sets (80/20)
    trainData, testData := base.InstancesTrainTestSplit(rawData, 0.80)
    
    fmt.Printf("Training samples: %d\n", trainData.Rows)
    fmt.Printf("Test samples: %d\n", testData.Rows)
}
```

### Classification: K-Nearest Neighbors

```go
func trainKNN() {
    // Load iris dataset
    rawData, _ := base.ParseCSVToInstances("iris.csv", true)
    trainData, testData := base.InstancesTrainTestSplit(rawData, 0.80)
    
    // Create KNN classifier (k=3)
    cls := knn.NewKnnClassifier("euclidean", "linear", 3)
    
    // Train
    cls.Fit(trainData)
    
    // Predict
    predictions, err := cls.Predict(testData)
    if err != nil {
        panic(err)
    }
    
    // Evaluate
    confusionMat, err := evaluation.GetConfusionMatrix(testData, predictions)
    if err != nil {
        panic(err)
    }
    
    fmt.Println(evaluation.GetSummary(confusionMat))
    
    // Accuracy
    accuracy := evaluation.GetAccuracy(confusionMat)
    fmt.Printf("Accuracy: %.2f%%\n", accuracy*100)
}
```

### Classification: Decision Tree

```go
import (
    "github.com/sjwhitworth/golearn/trees"
)

func trainDecisionTree() {
    rawData, _ := base.ParseCSVToInstances("data.csv", true)
    trainData, testData := base.InstancesTrainTestSplit(rawData, 0.70)
    
    // Create decision tree
    tree := trees.NewID3DecisionTree(0.6) // Pruning confidence
    
    // Train
    tree.Fit(trainData)
    
    // Predict
    predictions, _ := tree.Predict(testData)
    
    // Evaluate
    confusionMat, _ := evaluation.GetConfusionMatrix(testData, predictions)
    
    fmt.Println("Decision Tree Results:")
    fmt.Println(evaluation.GetSummary(confusionMat))
    
    // Precision, Recall, F1
    precision := evaluation.GetPrecision(confusionMat)
    recall := evaluation.GetRecall(confusionMat)
    f1 := evaluation.GetF1Score(confusionMat)
    
    fmt.Printf("Precision: %.2f\n", precision)
    fmt.Printf("Recall: %.2f\n", recall)
    fmt.Printf("F1 Score: %.2f\n", f1)
}
```

### Regression: Linear Regression

```go
import (
    "github.com/sjwhitworth/golearn/linear_models"
)

func trainLinearRegression() {
    // Load data
    rawData, _ := base.ParseCSVToInstances("housing.csv", true)
    trainData, testData := base.InstancesTrainTestSplit(rawData, 0.80)
    
    // Create linear regression model
    lr := linear_models.NewLinearRegression()
    
    // Train
    lr.Fit(trainData)
    
    // Predict
    predictions, _ := lr.Predict(testData)
    
    // Calculate Mean Squared Error
    mse := evaluation.GetMeanSquaredError(testData, predictions)
    rmse := math.Sqrt(mse)
    
    fmt.Printf("RMSE: %.2f\n", rmse)
    
    // Calculate R² score
    r2 := evaluation.GetRSquared(testData, predictions)
    fmt.Printf("R² Score: %.2f\n", r2)
}
```

### Clustering: K-Means

```go
import (
    "github.com/sjwhitworth/golearn/clustering"
)

func performKMeans() {
    // Load data
    rawData, _ := base.ParseCSVToInstances("data.csv", true)
    
    // Create K-Means clusterer (3 clusters)
    km := clustering.NewKMeansClusterer(3)
    
    // Fit
    km.Fit(rawData)
    
    // Get cluster assignments
    clusters := km.Predict(rawData)
    
    // Print cluster centers
    centers := km.GetClusterCenters()
    for i, center := range centers {
        fmt.Printf("Cluster %d center: %v\n", i, center)
    }
    
    // Count samples per cluster
    clusterCounts := make(map[int]int)
    for i := 0; i < clusters.Rows; i++ {
        cluster := int(clusters.Get(i, 0))
        clusterCounts[cluster]++
    }
    
    for cluster, count := range clusterCounts {
        fmt.Printf("Cluster %d: %d samples\n", cluster, count)
    }
}
```

### Cross-Validation

```go
func crossValidate() {
    rawData, _ := base.ParseCSVToInstances("data.csv", true)
    
    // Create classifier
    cls := knn.NewKnnClassifier("euclidean", "linear", 5)
    
    // 5-fold cross-validation
    cfs, err := evaluation.GenerateCrossFoldValidationConfusionMatrices(rawData, cls, 5)
    if err != nil {
        panic(err)
    }
    
    // Calculate mean accuracy
    mean, variance := evaluation.GetCrossValidatedMetric(cfs, evaluation.GetAccuracy)
    
    fmt.Printf("Mean Accuracy: %.2f%% (±%.2f%%)\n", mean*100, variance*100)
}
```

### Feature Engineering

```go
import (
    "github.com/sjwhitworth/golearn/filters"
)

func preprocessData(rawData base.FixedDataGrid) base.FixedDataGrid {
    // Remove attributes
    f := filters.NewRemoveAttributeFilter()
    f.AddAttribute(base.ResolveAttribute(rawData, "id"))
    f.AddAttribute(base.ResolveAttribute(rawData, "timestamp"))
    f.Train()
    result := base.NewLazilyFilteredInstances(rawData, f)
    
    // Discretize continuous attributes
    disc := filters.NewChiMergeFilter(rawData, 0.90)
    disc.AddAttribute(base.ResolveAttribute(rawData, "age"))
    disc.Train()
    result = base.NewLazilyFilteredInstances(result, disc)
    
    return result
}
```

---

## 14.3 Neural Networks with Gorgonia

### Gorgonia Fundamentals

Gorgonia is a library for building and training neural networks using computational graphs.

**Installation:**
```bash
go get -u gorgonia.org/gorgonia
go get -u gorgonia.org/tensor
```

### Computational Graph

```go
package main

import (
    "fmt"
    "log"
    
    . "gorgonia.org/gorgonia"
)

func main() {
    g := NewGraph()
    
    // Define variables
    x := NewScalar(g, Float64, WithName("x"))
    y := NewScalar(g, Float64, WithName("y"))
    
    // Define operations
    z := Must(Add(x, y))
    
    // Create VM
    machine := NewTapeMachine(g)
    defer machine.Close()
    
    // Set values
    Let(x, 2.0)
    Let(y, 3.0)
    
    // Run computation
    if err := machine.RunAll(); err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("%v + %v = %v\n", x.Value(), y.Value(), z.Value())
}
```

### Simple Neural Network (XOR Problem)

```go
package main

import (
    "fmt"
    "log"
    
    . "gorgonia.org/gorgonia"
    "gorgonia.org/tensor"
)

type NN struct {
    g      *ExprGraph
    w0, w1 *Node
    pred   *Node
}

func NewNN() *NN {
    g := NewGraph()
    
    // Input layer (2 inputs) → Hidden layer (2 neurons)
    w0 := NewMatrix(g, tensor.Float64, WithShape(2, 2), WithName("w0"), WithInit(GlorotU(1.0)))
    
    // Hidden layer (2 neurons) → Output layer (1 output)
    w1 := NewMatrix(g, tensor.Float64, WithShape(2, 1), WithName("w1"), WithInit(GlorotU(1.0)))
    
    return &NN{
        g:  g,
        w0: w0,
        w1: w1,
    }
}

func (nn *NN) fwd(x *Node) (err error) {
    var l0, l1 *Node
    
    // Layer 0: x · w0
    if l0, err = Mul(x, nn.w0); err != nil {
        return err
    }
    
    // Activation: sigmoid
    if l0, err = Sigmoid(l0); err != nil {
        return err
    }
    
    // Layer 1: l0 · w1
    if l1, err = Mul(l0, nn.w1); err != nil {
        return err
    }
    
    // Output activation: sigmoid
    if nn.pred, err = Sigmoid(l1); err != nil {
        return err
    }
    
    return nil
}

func main() {
    nn := NewNN()
    
    // XOR training data
    xorInputs := []float64{
        0, 0,
        0, 1,
        1, 0,
        1, 1,
    }
    xorTargets := []float64{0, 1, 1, 0}
    
    // Create input and target nodes
    x := NewMatrix(nn.g, tensor.Float64, WithShape(4, 2), WithName("x"))
    y := NewMatrix(nn.g, tensor.Float64, WithShape(4, 1), WithName("y"))
    
    // Forward pass
    nn.fwd(x)
    
    // Loss: Mean Squared Error
    losses := Must(Sub(y, nn.pred))
    square := Must(Square(losses))
    cost := Must(Mean(square))
    
    // Gradients
    if _, err := Grad(cost, nn.w0, nn.w1); err != nil {
        log.Fatal(err)
    }
    
    // Solver (learning rate)
    solver := NewVanillaSolver(WithLearnRate(0.1))
    
    // Create VM
    machine := NewTapeMachine(nn.g, BindDualValues(nn.w0, nn.w1))
    defer machine.Close()
    
    // Set input data
    xT := tensor.New(tensor.WithBacking(xorInputs), tensor.WithShape(4, 2))
    yT := tensor.New(tensor.WithBacking(xorTargets), tensor.WithShape(4, 1))
    Let(x, xT)
    Let(y, yT)
    
    // Training loop
    for i := 0; i < 10000; i++ {
        if err := machine.RunAll(); err != nil {
            log.Fatal(err)
        }
        
        solver.Step(NodesToValueGrads(nn.w0, nn.w1))
        machine.Reset()
        
        if i%1000 == 0 {
            fmt.Printf("Epoch %d: Cost = %v\n", i, cost.Value())
        }
    }
    
    // Test predictions
    fmt.Println("\nPredictions:")
    fmt.Printf("0 XOR 0 = %.4f (expected 0)\n", nn.pred.Value().Data().([]float64)[0])
    fmt.Printf("0 XOR 1 = %.4f (expected 1)\n", nn.pred.Value().Data().([]float64)[1])
    fmt.Printf("1 XOR 0 = %.4f (expected 1)\n", nn.pred.Value().Data().([]float64)[2])
    fmt.Printf("1 XOR 1 = %.4f (expected 0)\n", nn.pred.Value().Data().([]float64)[3])
}
```

### Convolutional Neural Network (CNN)

```go
type ConvNet struct {
    g          *ExprGraph
    conv1      *Node
    conv1b     *Node
    fc1        *Node
    fc1b       *Node
    out        *Node
    outb       *Node
    pred       *Node
}

func NewConvNet() *ConvNet {
    g := NewGraph()
    
    // Conv layer: 1 input channel, 32 filters, 3x3 kernel
    conv1 := NewTensor(g, tensor.Float64, 4, WithShape(32, 1, 3, 3), WithName("conv1"), WithInit(GlorotU(1.0)))
    conv1b := NewTensor(g, tensor.Float64, 1, WithShape(32), WithName("conv1b"), WithInit(Zeroes()))
    
    // Fully connected layer
    fc1 := NewMatrix(g, tensor.Float64, WithShape(800, 128), WithName("fc1"), WithInit(GlorotU(1.0)))
    fc1b := NewVector(g, tensor.Float64, WithShape(128), WithName("fc1b"), WithInit(Zeroes()))
    
    // Output layer
    out := NewMatrix(g, tensor.Float64, WithShape(128, 10), WithName("out"), WithInit(GlorotU(1.0)))
    outb := NewVector(g, tensor.Float64, WithShape(10), WithName("outb"), WithInit(Zeroes()))
    
    return &ConvNet{
        g:      g,
        conv1:  conv1,
        conv1b: conv1b,
        fc1:    fc1,
        fc1b:   fc1b,
        out:    out,
        outb:   outb,
    }
}

func (m *ConvNet) fwd(x *Node) (err error) {
    // Convolution + Bias + ReLU
    var c1 *Node
    if c1, err = Conv2d(x, m.conv1, tensor.Shape{3, 3}, []int{1, 1}, []int{1, 1}, []int{1, 1}); err != nil {
        return err
    }
    
    if c1, err = BroadcastAdd(c1, m.conv1b, nil, []byte{0, 2, 3}); err != nil {
        return err
    }
    
    if c1, err = Rectify(c1); err != nil {
        return err
    }
    
    // MaxPool
    if c1, err = MaxPool2D(c1, tensor.Shape{2, 2}, []int{0, 0}, []int{2, 2}); err != nil {
        return err
    }
    
    // Flatten
    var fc *Node
    b := x.Shape()[0]
    if fc, err = Reshape(c1, tensor.Shape{b, 800}); err != nil {
        return err
    }
    
    // Fully connected + ReLU
    if fc, err = Mul(fc, m.fc1); err != nil {
        return err
    }
    
    if fc, err = BroadcastAdd(fc, m.fc1b, nil, []byte{0}); err != nil {
        return err
    }
    
    if fc, err = Rectify(fc); err != nil {
        return err
    }
    
    // Output layer
    if m.pred, err = Mul(fc, m.out); err != nil {
        return err
    }
    
    if m.pred, err = BroadcastAdd(m.pred, m.outb, nil, []byte{0}); err != nil {
        return err
    }
    
    return nil
}
```

---

## 14.4 TensorFlow Integration

### TensorFlow Go Bindings

**Installation:**
```bash
# Install TensorFlow C library first
# See: https://www.tensorflow.org/install/lang_c

go get github.com/tensorflow/tensorflow/tensorflow/go
```

### Loading Pre-Trained Models

```go
package main

import (
    "fmt"
    "io/ioutil"
    
    tf "github.com/tensorflow/tensorflow/tensorflow/go"
)

func loadModel(modelPath string) (*tf.SavedModel, error) {
    // Load SavedModel
    model, err := tf.LoadSavedModel(modelPath, []string{"serve"}, nil)
    if err != nil {
        return nil, err
    }
    
    return model, nil
}

func predict(model *tf.SavedModel, inputData []float32) ([]float32, error) {
    // Create input tensor
    tensor, err := tf.NewTensor(inputData)
    if err != nil {
        return nil, err
    }
    
    // Run inference
    result, err := model.Session.Run(
        map[tf.Output]*tf.Tensor{
            model.Graph.Operation("input").Output(0): tensor,
        },
        []tf.Output{
            model.Graph.Operation("output").Output(0),
        },
        nil,
    )
    if err != nil {
        return nil, err
    }
    
    // Extract result
    predictions := result[0].Value().([][]float32)[0]
    return predictions, nil
}

func main() {
    model, err := loadModel("./saved_model")
    if err != nil {
        panic(err)
    }
    defer model.Session.Close()
    
    // Example input
    input := []float32{1.0, 2.0, 3.0, 4.0}
    
    predictions, err := predict(model, input)
    if err != nil {
        panic(err)
    }
    
    fmt.Printf("Predictions: %v\n", predictions)
}
```

### Image Classification

```go
func classifyImage(model *tf.SavedModel, imagePath string) (string, float32, error) {
    // Read and decode image
    imageData, err := ioutil.ReadFile(imagePath)
    if err != nil {
        return "", 0, err
    }
    
    // Decode JPEG/PNG
    tensor, err := tf.NewTensor(string(imageData))
    if err != nil {
        return "", 0, err
    }
    
    // Preprocess (normalize, resize, etc.)
    graph, input, output := makePreprocessGraph()
    session, err := tf.NewSession(graph, nil)
    if err != nil {
        return "", 0, err
    }
    defer session.Close()
    
    normalized, err := session.Run(
        map[tf.Output]*tf.Tensor{input: tensor},
        []tf.Output{output},
        nil,
    )
    if err != nil {
        return "", 0, err
    }
    
    // Run inference
    result, err := model.Session.Run(
        map[tf.Output]*tf.Tensor{
            model.Graph.Operation("input").Output(0): normalized[0],
        },
        []tf.Output{
            model.Graph.Operation("output").Output(0),
        },
        nil,
    )
    if err != nil {
        return "", 0, err
    }
    
    // Get top prediction
    probabilities := result[0].Value().([][]float32)[0]
    
    maxIdx := 0
    maxProb := probabilities[0]
    for i, prob := range probabilities {
        if prob > maxProb {
            maxProb = prob
            maxIdx = i
        }
    }
    
    label := getLabel(maxIdx)
    return label, maxProb, nil
}

func makePreprocessGraph() (*tf.Graph, tf.Output, tf.Output) {
    s := op.NewScope()
    input := op.Placeholder(s, tf.String)
    
    // Decode image
    decoded := op.DecodeJpeg(s, input, op.DecodeJpegChannels(3))
    
    // Convert to float and normalize
    floatImg := op.Cast(s, decoded, tf.Float)
    normalized := op.Div(s, floatImg, op.Const(s.SubScope("normalized"), float32(255.0)))
    
    // Resize to model input size (e.g., 224x224)
    resized := op.ResizeBilinear(s, 
        op.ExpandDims(s, normalized, op.Const(s.SubScope("axis"), int32(0))),
        op.Const(s.SubScope("size"), []int32{224, 224}))
    
    graph, _ := s.Finalize()
    return graph, input, resized
}
```

---

*I'll continue with ONNX Runtime, NLP, Computer Vision, Model Serving, and complete Part 14. Shall I continue?*

## 14.5 ONNX Runtime

### What is ONNX?

ONNX (Open Neural Network Exchange) is an open format for representing ML models, enabling interoperability between frameworks.

**Benefits:**
- Train in PyTorch/TensorFlow, serve in Go
- Framework-independent deployment
- Optimized inference performance
- Hardware acceleration support

**Installation:**
```bash
go get github.com/yalue/onnxruntime_go
```

### Loading ONNX Models

```go
package main

import (
    "fmt"
    
    ort "github.com/yalue/onnxruntime_go"
)

func main() {
    // Initialize ONNX Runtime
    ort.SetSharedLibraryPath("/path/to/onnxruntime.so")
    err := ort.InitializeEnvironment()
    if err != nil {
        panic(err)
    }
    defer ort.DestroyEnvironment()
    
    // Load model
    session, err := ort.NewSession[float32]("model.onnx",
        []string{"input"},
        []string{"output"},
        [][]int64{{1, 3, 224, 224}}, // Input shape
        [][]int64{{1, 1000}},         // Output shape
    )
    if err != nil {
        panic(err)
    }
    defer session.Destroy()
    
    // Prepare input
    inputData := make([]float32, 1*3*224*224)
    // ... fill inputData with image data
    
    // Run inference
    outputs, err := session.Run([][]float32{inputData})
    if err != nil {
        panic(err)
    }
    
    fmt.Printf("Output: %v\n", outputs[0][:10]) // First 10 predictions
}
```

### Image Classification with ONNX

```go
type ImageClassifier struct {
    session    *ort.Session[float32]
    inputShape []int64
    labels     []string
}

func NewImageClassifier(modelPath string, labelsPath string) (*ImageClassifier, error) {
    // Initialize ONNX Runtime
    if err := ort.InitializeEnvironment(); err != nil {
        return nil, err
    }
    
    // Load model
    session, err := ort.NewSession[float32](
        modelPath,
        []string{"input"},
        []string{"output"},
        [][]int64{{1, 3, 224, 224}},
        [][]int64{{1, 1000}},
    )
    if err != nil {
        return nil, err
    }
    
    // Load labels
    labels, err := loadLabels(labelsPath)
    if err != nil {
        return nil, err
    }
    
    return &ImageClassifier{
        session:    session,
        inputShape: []int64{1, 3, 224, 224},
        labels:     labels,
    }, nil
}

func (ic *ImageClassifier) Classify(imageData []float32) (string, float32, error) {
    // Run inference
    outputs, err := ic.session.Run([][]float32{imageData})
    if err != nil {
        return "", 0, err
    }
    
    // Find top prediction
    predictions := outputs[0]
    maxIdx := 0
    maxScore := predictions[0]
    
    for i, score := range predictions {
        if score > maxScore {
            maxScore = score
            maxIdx = i
        }
    }
    
    label := ic.labels[maxIdx]
    return label, maxScore, nil
}

func (ic *ImageClassifier) Close() {
    ic.session.Destroy()
    ort.DestroyEnvironment()
}
```

### Model Quantization

```go
// Quantized model (INT8) for faster inference
func loadQuantizedModel(modelPath string) (*ort.Session[int8], error) {
    session, err := ort.NewSession[int8](
        modelPath,
        []string{"input"},
        []string{"output"},
        [][]int64{{1, 3, 224, 224}},
        [][]int64{{1, 1000}},
    )
    
    return session, err
}

// Quantize float32 input to int8
func quantize(data []float32, scale float32, zeroPoint int8) []int8 {
    quantized := make([]int8, len(data))
    for i, v := range data {
        quantized[i] = int8(v/scale) + zeroPoint
    }
    return quantized
}

// Dequantize int8 output to float32
func dequantize(data []int8, scale float32, zeroPoint int8) []float32 {
    dequantized := make([]float32, len(data))
    for i, v := range data {
        dequantized[i] = float32(v-zeroPoint) * scale
    }
    return dequantized
}
```

---

## 14.6 Natural Language Processing

### Text Processing with prose

```bash
go get github.com/jdkato/prose/v2
```

**Tokenization:**
```go
package main

import (
    "fmt"
    
    "github.com/jdkato/prose/v2"
)

func main() {
    doc, _ := prose.NewDocument("Go is a great programming language. It's fast and simple!")
    
    // Tokenize
    for _, tok := range doc.Tokens() {
        fmt.Printf("%s ", tok.Text)
    }
    fmt.Println()
    
    // Sentences
    for _, sent := range doc.Sentences() {
        fmt.Println(sent.Text)
    }
}
```

**Part-of-Speech Tagging:**
```go
func posTagging(text string) {
    doc, _ := prose.NewDocument(text)
    
    for _, tok := range doc.Tokens() {
        fmt.Printf("%s/%s ", tok.Text, tok.Tag)
    }
    fmt.Println()
}

// Output: "The/DT cat/NN sat/VBD on/IN the/DT mat/NN"
```

**Named Entity Recognition:**
```go
func extractEntities(text string) {
    doc, _ := prose.NewDocument(text)
    
    for _, ent := range doc.Entities() {
        fmt.Printf("%s (%s)\n", ent.Text, ent.Label)
    }
}

// Example: "Apple" (ORGANIZATION), "Tim Cook" (PERSON)
```

### Sentiment Analysis

```go
type SentimentAnalyzer struct {
    positiveWords map[string]bool
    negativeWords map[string]bool
}

func NewSentimentAnalyzer() *SentimentAnalyzer {
    return &SentimentAnalyzer{
        positiveWords: loadWordList("positive.txt"),
        negativeWords: loadWordList("negative.txt"),
    }
}

func (sa *SentimentAnalyzer) Analyze(text string) float64 {
    doc, _ := prose.NewDocument(text)
    
    positive := 0
    negative := 0
    
    for _, tok := range doc.Tokens() {
        word := strings.ToLower(tok.Text)
        
        if sa.positiveWords[word] {
            positive++
        }
        if sa.negativeWords[word] {
            negative++
        }
    }
    
    total := positive + negative
    if total == 0 {
        return 0.0 // Neutral
    }
    
    // Return score from -1 (negative) to +1 (positive)
    return float64(positive-negative) / float64(total)
}

func loadWordList(filename string) map[string]bool {
    words := make(map[string]bool)
    
    file, _ := os.Open(filename)
    defer file.Close()
    
    scanner := bufio.NewScanner(file)
    for scanner.Scan() {
        words[scanner.Text()] = true
    }
    
    return words
}
```

### TF-IDF (Term Frequency-Inverse Document Frequency)

```go
type TFIDF struct {
    documents [][]string
    idf       map[string]float64
}

func NewTFIDF(documents []string) *TFIDF {
    tfidf := &TFIDF{
        documents: make([][]string, len(documents)),
        idf:       make(map[string]float64),
    }
    
    // Tokenize documents
    for i, doc := range documents {
        tfidf.documents[i] = tokenize(doc)
    }
    
    // Calculate IDF
    tfidf.calculateIDF()
    
    return tfidf
}

func (t *TFIDF) calculateIDF() {
    docCount := float64(len(t.documents))
    termDocCount := make(map[string]int)
    
    // Count documents containing each term
    for _, doc := range t.documents {
        seen := make(map[string]bool)
        for _, term := range doc {
            if !seen[term] {
                termDocCount[term]++
                seen[term] = true
            }
        }
    }
    
    // Calculate IDF
    for term, count := range termDocCount {
        t.idf[term] = math.Log(docCount / float64(count))
    }
}

func (t *TFIDF) GetTFIDF(docIndex int) map[string]float64 {
    doc := t.documents[docIndex]
    tfIdf := make(map[string]float64)
    
    // Calculate TF
    termFreq := make(map[string]int)
    for _, term := range doc {
        termFreq[term]++
    }
    
    docLen := float64(len(doc))
    
    // Calculate TF-IDF
    for term, freq := range termFreq {
        tf := float64(freq) / docLen
        tfIdf[term] = tf * t.idf[term]
    }
    
    return tfIdf
}

func tokenize(text string) []string {
    // Simple tokenization (whitespace split + lowercase)
    text = strings.ToLower(text)
    return strings.Fields(text)
}
```

### Text Classification

```go
type TextClassifier struct {
    model     *golearn.Classifier
    vectorizer *TFIDFVectorizer
}

type TFIDFVectorizer struct {
    vocabulary map[string]int
    idf        []float64
}

func (v *TFIDFVectorizer) Transform(text string) []float64 {
    tokens := tokenize(text)
    vector := make([]float64, len(v.vocabulary))
    
    // Calculate TF
    termFreq := make(map[string]int)
    for _, token := range tokens {
        termFreq[token]++
    }
    
    // Calculate TF-IDF
    for term, freq := range termFreq {
        if idx, exists := v.vocabulary[term]; exists {
            tf := float64(freq) / float64(len(tokens))
            vector[idx] = tf * v.idf[idx]
        }
    }
    
    return vector
}

func (tc *TextClassifier) Predict(text string) string {
    features := tc.vectorizer.Transform(text)
    prediction := tc.model.Predict(features)
    return prediction
}
```

---

## 14.7 Computer Vision with gocv

### gocv (OpenCV Bindings)

```bash
# Install OpenCV first
# See: https://gocv.io/getting-started/

go get -u gocv.io/x/gocv
```

### Image Loading and Processing

```go
package main

import (
    "fmt"
    
    "gocv.io/x/gocv"
)

func main() {
    // Read image
    img := gocv.IMRead("image.jpg", gocv.IMReadColor)
    if img.Empty() {
        panic("Error reading image")
    }
    defer img.Close()
    
    fmt.Printf("Image size: %dx%d\n", img.Cols(), img.Rows())
    
    // Convert to grayscale
    gray := gocv.NewMat()
    defer gray.Close()
    gocv.CvtColor(img, &gray, gocv.ColorBGRToGray)
    
    // Save result
    gocv.IMWrite("gray.jpg", gray)
}
```

### Image Transformations

```go
func imageTransformations(img gocv.Mat) {
    // Resize
    resized := gocv.NewMat()
    defer resized.Close()
    gocv.Resize(img, &resized, image.Point{X: 640, Y: 480}, 0, 0, gocv.InterpolationLinear)
    
    // Rotate
    rotated := gocv.NewMat()
    defer rotated.Close()
    center := image.Point{X: img.Cols() / 2, Y: img.Rows() / 2}
    rotMatrix := gocv.GetRotationMatrix2D(center, 45, 1.0)
    gocv.WarpAffine(img, &rotated, rotMatrix, image.Point{X: img.Cols(), Y: img.Rows()})
    
    // Flip
    flipped := gocv.NewMat()
    defer flipped.Close()
    gocv.Flip(img, &flipped, 1) // 1 = horizontal, 0 = vertical, -1 = both
    
    // Crop
    rect := image.Rect(100, 100, 300, 300)
    cropped := img.Region(rect)
    defer cropped.Close()
}
```

### Filters and Effects

```go
func applyFilters(img gocv.Mat) {
    // Gaussian blur
    blurred := gocv.NewMat()
    defer blurred.Close()
    gocv.GaussianBlur(img, &blurred, image.Point{X: 5, Y: 5}, 0, 0, gocv.BorderDefault)
    
    // Edge detection (Canny)
    edges := gocv.NewMat()
    defer edges.Close()
    gocv.Canny(img, &edges, 50, 150)
    
    // Threshold
    binary := gocv.NewMat()
    defer binary.Close()
    gocv.Threshold(img, &binary, 127, 255, gocv.ThresholdBinary)
    
    // Morphological operations
    kernel := gocv.GetStructuringElement(gocv.MorphRect, image.Point{X: 5, Y: 5})
    defer kernel.Close()
    
    dilated := gocv.NewMat()
    defer dilated.Close()
    gocv.Dilate(img, &dilated, kernel)
    
    eroded := gocv.NewMat()
    defer eroded.Close()
    gocv.Erode(img, &eroded, kernel)
}
```

### Object Detection (Cascade Classifier)

```go
func detectFaces(img gocv.Mat) []image.Rectangle {
    // Load cascade classifier
    classifier := gocv.NewCascadeClassifier()
    defer classifier.Close()
    
    if !classifier.Load("haarcascade_frontalface_default.xml") {
        panic("Error loading cascade")
    }
    
    // Detect faces
    faces := classifier.DetectMultiScale(img)
    
    // Draw rectangles around faces
    for _, face := range faces {
        gocv.Rectangle(&img, face, color.RGBA{0, 255, 0, 0}, 2)
    }
    
    return faces
}
```

### Video Processing

```go
func processVideo(filename string) {
    // Open video
    video, err := gocv.VideoCaptureFile(filename)
    if err != nil {
        panic(err)
    }
    defer video.Close()
    
    // Create window
    window := gocv.NewWindow("Video")
    defer window.Close()
    
    img := gocv.NewMat()
    defer img.Close()
    
    // Process frames
    for {
        if ok := video.Read(&img); !ok {
            break
        }
        if img.Empty() {
            continue
        }
        
        // Process frame (e.g., apply filter)
        processFrame(&img)
        
        // Display
        window.IMShow(img)
        if window.WaitKey(1) >= 0 {
            break
        }
    }
}

func processFrame(img *gocv.Mat) {
    // Apply processing (edge detection, face detection, etc.)
    gocv.Canny(*img, img, 50, 150)
}
```

### Webcam Capture

```go
func captureWebcam() {
    // Open webcam (0 = default camera)
    webcam, err := gocv.VideoCaptureDevice(0)
    if err != nil {
        panic(err)
    }
    defer webcam.Close()
    
    window := gocv.NewWindow("Webcam")
    defer window.Close()
    
    img := gocv.NewMat()
    defer img.Close()
    
    for {
        if ok := webcam.Read(&img); !ok {
            continue
        }
        if img.Empty() {
            continue
        }
        
        window.IMShow(img)
        if window.WaitKey(1) >= 0 {
            break
        }
    }
}
```

---

## 14.8 Model Serving and Deployment

### REST API for Model Serving

```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
    
    ort "github.com/yalue/onnxruntime_go"
)

type PredictionRequest struct {
    Features []float32 `json:"features"`
}

type PredictionResponse struct {
    Predictions []float32 `json:"predictions"`
    Class       int       `json:"class"`
    Confidence  float32   `json:"confidence"`
}

type ModelServer struct {
    session *ort.Session[float32]
}

func NewModelServer(modelPath string) (*ModelServer, error) {
    ort.InitializeEnvironment()
    
    session, err := ort.NewSession[float32](
        modelPath,
        []string{"input"},
        []string{"output"},
        [][]int64{{1, 784}},  // MNIST example
        [][]int64{{1, 10}},
    )
    if err != nil {
        return nil, err
    }
    
    return &ModelServer{session: session}, nil
}

func (ms *ModelServer) PredictHandler(w http.ResponseWriter, r *http.Request) {
    var req PredictionRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    // Run inference
    outputs, err := ms.session.Run([][]float32{req.Features})
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    
    predictions := outputs[0]
    
    // Find predicted class
    maxIdx := 0
    maxConf := predictions[0]
    for i, p := range predictions {
        if p > maxConf {
            maxConf = p
            maxIdx = i
        }
    }
    
    resp := PredictionResponse{
        Predictions: predictions,
        Class:       maxIdx,
        Confidence:  maxConf,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(resp)
}

func main() {
    server, err := NewModelServer("model.onnx")
    if err != nil {
        log.Fatal(err)
    }
    
    http.HandleFunc("/predict", server.PredictHandler)
    http.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        w.Write([]byte("OK"))
    })
    
    log.Println("Model server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### Batch Prediction API

```go
type BatchPredictionRequest struct {
    Instances [][]float32 `json:"instances"`
}

type BatchPredictionResponse struct {
    Predictions [][]float32 `json:"predictions"`
}

func (ms *ModelServer) BatchPredictHandler(w http.ResponseWriter, r *http.Request) {
    var req BatchPredictionRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    var allPredictions [][]float32
    
    // Process in batches
    batchSize := 32
    for i := 0; i < len(req.Instances); i += batchSize {
        end := i + batchSize
        if end > len(req.Instances) {
            end = len(req.Instances)
        }
        
        batch := req.Instances[i:end]
        
        // Flatten batch
        var flatBatch []float32
        for _, instance := range batch {
            flatBatch = append(flatBatch, instance...)
        }
        
        // Run inference
        outputs, err := ms.session.Run([][]float32{flatBatch})
        if err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }
        
        // Reshape outputs
        for j := 0; j < len(batch); j++ {
            start := j * 10 // 10 classes
            end := start + 10
            allPredictions = append(allPredictions, outputs[0][start:end])
        }
    }
    
    resp := BatchPredictionResponse{
        Predictions: allPredictions,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(resp)
}
```

### Model Versioning

```go
type ModelRegistry struct {
    models map[string]*ort.Session[float32]
    mu     sync.RWMutex
}

func NewModelRegistry() *ModelRegistry {
    return &ModelRegistry{
        models: make(map[string]*ort.Session[float32]),
    }
}

func (mr *ModelRegistry) LoadModel(version string, path string) error {
    session, err := ort.NewSession[float32](
        path,
        []string{"input"},
        []string{"output"},
        [][]int64{{1, 784}},
        [][]int64{{1, 10}},
    )
    if err != nil {
        return err
    }
    
    mr.mu.Lock()
    mr.models[version] = session
    mr.mu.Unlock()
    
    return nil
}

func (mr *ModelRegistry) GetModel(version string) (*ort.Session[float32], error) {
    mr.mu.RLock()
    defer mr.mu.RUnlock()
    
    model, exists := mr.models[version]
    if !exists {
        return nil, fmt.Errorf("model version %s not found", version)
    }
    
    return model, nil
}

func (mr *ModelRegistry) PredictWithVersion(version string, input []float32) ([]float32, error) {
    model, err := mr.GetModel(version)
    if err != nil {
        return nil, err
    }
    
    outputs, err := model.Run([][]float32{input})
    if err != nil {
        return nil, err
    }
    
    return outputs[0], nil
}
```

---

*I'll complete Part 14 with AI API Integration, FAQs, Interview Questions, and exercises. Shall I continue?*

## 14.9 AI API Integration

### OpenAI API

```bash
go get github.com/sashabaranov/go-openai
```

**Chat Completions:**
```go
package main

import (
    "context"
    "fmt"
    
    openai "github.com/sashabaranov/go-openai"
)

func chatCompletion(apiKey, prompt string) (string, error) {
    client := openai.NewClient(apiKey)
    
    resp, err := client.CreateChatCompletion(
        context.Background(),
        openai.ChatCompletionRequest{
            Model: openai.GPT4,
            Messages: []openai.ChatCompletionMessage{
                {
                    Role:    openai.ChatMessageRoleUser,
                    Content: prompt,
                },
            },
        },
    )
    
    if err != nil {
        return "", err
    }
    
    return resp.Choices[0].Message.Content, nil
}

func main() {
    apiKey := "your-api-key"
    prompt := "Explain machine learning in simple terms"
    
    response, err := chatCompletion(apiKey, prompt)
    if err != nil {
        panic(err)
    }
    
    fmt.Println(response)
}
```

**Streaming Responses:**
```go
func streamChatCompletion(apiKey, prompt string) error {
    client := openai.NewClient(apiKey)
    
    stream, err := client.CreateChatCompletionStream(
        context.Background(),
        openai.ChatCompletionRequest{
            Model: openai.GPT4,
            Messages: []openai.ChatCompletionMessage{
                {
                    Role:    openai.ChatMessageRoleUser,
                    Content: prompt,
                },
            },
            Stream: true,
        },
    )
    if err != nil {
        return err
    }
    defer stream.Close()
    
    fmt.Print("Response: ")
    for {
        response, err := stream.Recv()
        if errors.Is(err, io.EOF) {
            fmt.Println()
            break
        }
        if err != nil {
            return err
        }
        
        fmt.Print(response.Choices[0].Delta.Content)
    }
    
    return nil
}
```

**Embeddings:**
```go
func getEmbeddings(apiKey string, texts []string) ([][]float32, error) {
    client := openai.NewClient(apiKey)
    
    resp, err := client.CreateEmbeddings(
        context.Background(),
        openai.EmbeddingRequest{
            Model: openai.AdaEmbeddingV2,
            Input: texts,
        },
    )
    if err != nil {
        return nil, err
    }
    
    embeddings := make([][]float32, len(resp.Data))
    for i, data := range resp.Data {
        embeddings[i] = data.Embedding
    }
    
    return embeddings, nil
}

// Semantic search using embeddings
func semanticSearch(query string, documents []string, apiKey string) (int, error) {
    // Get embeddings
    allTexts := append([]string{query}, documents...)
    embeddings, err := getEmbeddings(apiKey, allTexts)
    if err != nil {
        return -1, err
    }
    
    queryEmb := embeddings[0]
    docEmbs := embeddings[1:]
    
    // Find most similar document (cosine similarity)
    maxSim := -1.0
    maxIdx := -1
    
    for i, docEmb := range docEmbs {
        sim := cosineSimilarity(queryEmb, docEmb)
        if sim > maxSim {
            maxSim = sim
            maxIdx = i
        }
    }
    
    return maxIdx, nil
}

func cosineSimilarity(a, b []float32) float64 {
    var dotProduct, normA, normB float64
    
    for i := range a {
        dotProduct += float64(a[i] * b[i])
        normA += float64(a[i] * a[i])
        normB += float64(b[i] * b[i])
    }
    
    return dotProduct / (math.Sqrt(normA) * math.Sqrt(normB))
}
```

**Image Generation (DALL-E):**
```go
func generateImage(apiKey, prompt string) (string, error) {
    client := openai.NewClient(apiKey)
    
    resp, err := client.CreateImage(
        context.Background(),
        openai.ImageRequest{
            Prompt: prompt,
            N:      1,
            Size:   openai.CreateImageSize1024x1024,
        },
    )
    if err != nil {
        return "", err
    }
    
    return resp.Data[0].URL, nil
}
```

### Anthropic Claude API

```go
type ClaudeClient struct {
    apiKey  string
    baseURL string
    client  *http.Client
}

type ClaudeRequest struct {
    Model     string    `json:"model"`
    Messages  []Message `json:"messages"`
    MaxTokens int       `json:"max_tokens"`
}

type Message struct {
    Role    string `json:"role"`
    Content string `json:"content"`
}

type ClaudeResponse struct {
    ID      string `json:"id"`
    Content []struct {
        Text string `json:"text"`
    } `json:"content"`
}

func NewClaudeClient(apiKey string) *ClaudeClient {
    return &ClaudeClient{
        apiKey:  apiKey,
        baseURL: "https://api.anthropic.com/v1/messages",
        client:  &http.Client{Timeout: 60 * time.Second},
    }
}

func (c *ClaudeClient) SendMessage(prompt string) (string, error) {
    reqBody := ClaudeRequest{
        Model: "claude-3-opus-20240229",
        Messages: []Message{
            {Role: "user", Content: prompt},
        },
        MaxTokens: 1024,
    }
    
    jsonData, _ := json.Marshal(reqBody)
    
    req, _ := http.NewRequest("POST", c.baseURL, bytes.NewBuffer(jsonData))
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("x-api-key", c.apiKey)
    req.Header.Set("anthropic-version", "2023-06-01")
    
    resp, err := c.client.Do(req)
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()
    
    var claudeResp ClaudeResponse
    if err := json.NewDecoder(resp.Body).Decode(&claudeResp); err != nil {
        return "", err
    }
    
    if len(claudeResp.Content) == 0 {
        return "", fmt.Errorf("no response content")
    }
    
    return claudeResp.Content[0].Text, nil
}
```

### Hugging Face API

```go
type HuggingFaceClient struct {
    apiKey string
    client *http.Client
}

func (hf *HuggingFaceClient) Inference(modelID string, inputs map[string]interface{}) (map[string]interface{}, error) {
    url := fmt.Sprintf("https://api-inference.huggingface.co/models/%s", modelID)
    
    jsonData, _ := json.Marshal(inputs)
    
    req, _ := http.NewRequest("POST", url, bytes.NewBuffer(jsonData))
    req.Header.Set("Authorization", "Bearer "+hf.apiKey)
    req.Header.Set("Content-Type", "application/json")
    
    resp, err := hf.client.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var result map[string]interface{}
    if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
        return nil, err
    }
    
    return result, nil
}

// Text classification example
func (hf *HuggingFaceClient) ClassifyText(text string) (string, float64, error) {
    result, err := hf.Inference("distilbert-base-uncased-finetuned-sst-2-english", map[string]interface{}{
        "inputs": text,
    })
    if err != nil {
        return "", 0, err
    }
    
    // Parse result
    predictions := result["predictions"].([]interface{})
    if len(predictions) == 0 {
        return "", 0, fmt.Errorf("no predictions")
    }
    
    pred := predictions[0].(map[string]interface{})
    label := pred["label"].(string)
    score := pred["score"].(float64)
    
    return label, score, nil
}
```

---

## 14.10 MLOps Patterns

### Model Monitoring

```go
type ModelMonitor struct {
    predictionCount   prometheus.Counter
    predictionLatency prometheus.Histogram
    predictionErrors  prometheus.Counter
    inputDrift        prometheus.Gauge
}

func NewModelMonitor() *ModelMonitor {
    return &ModelMonitor{
        predictionCount: promauto.NewCounter(prometheus.CounterOpts{
            Name: "model_predictions_total",
            Help: "Total number of predictions",
        }),
        predictionLatency: promauto.NewHistogram(prometheus.HistogramOpts{
            Name:    "model_prediction_duration_seconds",
            Help:    "Prediction latency",
            Buckets: prometheus.DefBuckets,
        }),
        predictionErrors: promauto.NewCounter(prometheus.CounterOpts{
            Name: "model_prediction_errors_total",
            Help: "Total prediction errors",
        }),
        inputDrift: promauto.NewGauge(prometheus.GaugeOpts{
            Name: "model_input_drift",
            Help: "Input data drift metric",
        }),
    }
}

func (m *ModelMonitor) RecordPrediction(duration time.Duration, err error) {
    m.predictionCount.Inc()
    m.predictionLatency.Observe(duration.Seconds())
    
    if err != nil {
        m.predictionErrors.Inc()
    }
}

func (m *ModelMonitor) UpdateDriftMetric(drift float64) {
    m.inputDrift.Set(drift)
}
```

### Feature Store

```go
type FeatureStore struct {
    redis *redis.Client
}

func NewFeatureStore(redisAddr string) *FeatureStore {
    return &FeatureStore{
        redis: redis.NewClient(&redis.Options{
            Addr: redisAddr,
        }),
    }
}

func (fs *FeatureStore) GetFeatures(entityID string, featureNames []string) (map[string]interface{}, error) {
    features := make(map[string]interface{})
    
    for _, name := range featureNames {
        key := fmt.Sprintf("feature:%s:%s", entityID, name)
        val, err := fs.redis.Get(context.Background(), key).Result()
        if err != nil {
            return nil, err
        }
        
        features[name] = val
    }
    
    return features, nil
}

func (fs *FeatureStore) SetFeature(entityID, featureName string, value interface{}, ttl time.Duration) error {
    key := fmt.Sprintf("feature:%s:%s", entityID, featureName)
    return fs.redis.Set(context.Background(), key, value, ttl).Err()
}

// Batch feature retrieval
func (fs *FeatureStore) GetBatchFeatures(entityIDs []string, featureNames []string) (map[string]map[string]interface{}, error) {
    result := make(map[string]map[string]interface{})
    
    pipe := fs.redis.Pipeline()
    cmds := make(map[string]*redis.StringCmd)
    
    for _, entityID := range entityIDs {
        for _, name := range featureNames {
            key := fmt.Sprintf("feature:%s:%s", entityID, name)
            cmds[key] = pipe.Get(context.Background(), key)
        }
    }
    
    _, err := pipe.Exec(context.Background())
    if err != nil && err != redis.Nil {
        return nil, err
    }
    
    for key, cmd := range cmds {
        parts := strings.Split(key, ":")
        entityID := parts[1]
        featureName := parts[2]
        
        val, _ := cmd.Result()
        
        if _, exists := result[entityID]; !exists {
            result[entityID] = make(map[string]interface{})
        }
        result[entityID][featureName] = val
    }
    
    return result, nil
}
```

### A/B Testing for Models

```go
type ModelExperiment struct {
    modelA      *ort.Session[float32]
    modelB      *ort.Session[float32]
    trafficSplit float64 // 0.0 to 1.0 (% to model B)
    metrics     *ExperimentMetrics
}

type ExperimentMetrics struct {
    modelACount   int
    modelBCount   int
    modelALatency []time.Duration
    modelBLatency []time.Duration
    mu            sync.Mutex
}

func (me *ModelExperiment) Predict(input []float32) ([]float32, string, error) {
    // Random traffic split
    useModelB := rand.Float64() < me.trafficSplit
    
    var output []float32
    var err error
    var modelName string
    
    start := time.Now()
    
    if useModelB {
        outputs, err := me.modelB.Run([][]float32{input})
        if err == nil {
            output = outputs[0]
        }
        modelName = "model_b"
        
        me.metrics.mu.Lock()
        me.metrics.modelBCount++
        me.metrics.modelBLatency = append(me.metrics.modelBLatency, time.Since(start))
        me.metrics.mu.Unlock()
    } else {
        outputs, err := me.modelA.Run([][]float32{input})
        if err == nil {
            output = outputs[0]
        }
        modelName = "model_a"
        
        me.metrics.mu.Lock()
        me.metrics.modelACount++
        me.metrics.modelALatency = append(me.metrics.modelALatency, time.Since(start))
        me.metrics.mu.Unlock()
    }
    
    return output, modelName, err
}

func (me *ModelExperiment) GetMetrics() map[string]interface{} {
    me.metrics.mu.Lock()
    defer me.metrics.mu.Unlock()
    
    avgLatencyA := calculateAverage(me.metrics.modelALatency)
    avgLatencyB := calculateAverage(me.metrics.modelBLatency)
    
    return map[string]interface{}{
        "model_a_count":      me.metrics.modelACount,
        "model_b_count":      me.metrics.modelBCount,
        "model_a_avg_latency": avgLatencyA.Milliseconds(),
        "model_b_avg_latency": avgLatencyB.Milliseconds(),
    }
}

func calculateAverage(durations []time.Duration) time.Duration {
    if len(durations) == 0 {
        return 0
    }
    
    var total time.Duration
    for _, d := range durations {
        total += d
    }
    
    return total / time.Duration(len(durations))
}
```

---

## FAQs

**Q1: Should I use Go for machine learning?**

**For training:** No, use Python (richer ecosystem, better GPU support, more libraries)  
**For serving/deployment:** Yes! Go excels at:
- Fast inference servers
- Low latency predictions
- Concurrent request handling
- Easy deployment (single binary)
- Production infrastructure

**Q2: How do I convert a Python-trained model to Go?**

Export to ONNX format:
```python
# In Python
import torch
torch.onnx.export(model, dummy_input, "model.onnx")
```

Then load in Go with ONNX Runtime (see section 14.5).

**Q3: What's the difference between GoLearn and Gorgonia?**

- **GoLearn**: Traditional ML (classification, regression, clustering) - like scikit-learn
- **Gorgonia**: Deep learning (neural networks) - like PyTorch/TensorFlow

**Q4: How do I handle GPU acceleration in Go?**

Options:
1. Use TensorFlow/ONNX Runtime (C bindings with GPU support)
2. Use Gorgonia with CUDA backend (experimental)
3. Call Python models via API (train/serve in Python)

**Q5: What's the best approach for production ML with Go?**

**Recommended pattern:**
1. Train models in Python (TensorFlow, PyTorch)
2. Export to ONNX
3. Serve with Go (ONNX Runtime)
4. Monitor with Prometheus
5. Version models properly

This gives you the best of both worlds!

---

## Interview Questions

**Q1: Explain the difference between classification and regression.**

**A:** 
- **Classification**: Predict discrete categories (spam/not spam, cat/dog)
- **Regression**: Predict continuous values (house price, temperature)

Both are supervised learning but differ in output type.

**Q2: What is overfitting and how do you prevent it?**

**A:** Overfitting is when a model learns training data too well, including noise, and performs poorly on new data.

**Prevention:**
- Cross-validation
- Regularization (L1/L2)
- More training data
- Simpler models
- Early stopping
- Dropout (neural networks)

**Q3: How would you deploy a machine learning model in production?**

**A:**
1. Export model (ONNX, SavedModel)
2. Create inference service (REST/gRPC API)
3. Add monitoring (latency, errors, drift)
4. Version models
5. Implement A/B testing
6. Set up CI/CD pipeline
7. Monitor performance metrics
8. Plan for model updates

**Q4: Explain the bias-variance tradeoff.**

**A:**
- **High bias**: Model is too simple, underfits data
- **High variance**: Model is too complex, overfits data
- **Goal**: Find balance for good generalization

Simple models → high bias, low variance  
Complex models → low bias, high variance

**Q5: What is the purpose of embeddings in NLP?**

**A:** Embeddings convert words/text into dense numerical vectors that capture semantic meaning. Words with similar meanings have similar vectors.

**Benefits:**
- Reduce dimensionality
- Capture semantic relationships
- Enable similarity comparisons
- Work with neural networks

Example: word2vec, GloVe, BERT embeddings

---

## Key Takeaways

1. **Go for inference**, Python for training (best practice)
2. **GoLearn** for traditional ML, **Gorgonia** for deep learning
3. **ONNX** enables framework-independent model deployment
4. **TensorFlow Go bindings** work but require C dependencies
5. **Computer vision** possible with gocv (OpenCV)
6. **NLP** supported with prose and custom implementations
7. **Model serving** in Go is fast, concurrent, and production-ready
8. **AI APIs** (OpenAI, Claude) integrate easily for powerful features
9. **MLOps** patterns (monitoring, A/B testing) are crucial for production

---

## Practice Exercises

### Exercise 1: Build Image Classifier
- Load pre-trained ONNX model
- Create REST API for image classification
- Add preprocessing (resize, normalize)
- Return top-5 predictions with confidence scores

### Exercise 2: Sentiment Analysis Service
- Implement TF-IDF vectorization
- Train classifier with GoLearn
- Create API endpoint
- Add caching for common phrases

### Exercise 3: Model A/B Testing Platform
- Load two model versions
- Implement traffic splitting
- Track metrics for each model
- Create dashboard to compare performance

### Exercise 4: Feature Store
- Design feature storage schema
- Implement Redis-backed feature store
- Add batch retrieval
- Create API for feature access

### Exercise 5: AI Chatbot with OpenAI
- Integrate OpenAI API
- Implement conversation history
- Add streaming responses
- Create web interface

---

## Additional Resources

**Machine Learning:**
- Hands-On Machine Learning (Aurélien Géron) - Book
- https://www.coursera.org/learn/machine-learning (Andrew Ng)

**Go + ML:**
- https://github.com/sjwhitworth/golearn
- https://gorgonia.org
- https://github.com/onnx/onnx

**ONNX:**
- https://onnx.ai
- https://github.com/microsoft/onnxruntime

**Computer Vision:**
- https://gocv.io
- Learning OpenCV (O'Reilly) - Book

**NLP:**
- https://github.com/jdkato/prose
- Speech and Language Processing (Jurafsky) - Book

**MLOps:**
- https://ml-ops.org
- Introducing MLOps (O'Reilly) - Book

---

## Summary

Part 14 covered machine learning and AI integration with Go:

1. **ML Fundamentals**: When to use Go for ML
2. **GoLearn**: Classification, regression, clustering
3. **Gorgonia**: Neural networks, computational graphs
4. **TensorFlow**: Model loading and inference
5. **ONNX Runtime**: Framework-independent deployment
6. **NLP**: Text processing, sentiment analysis, TF-IDF
7. **Computer Vision**: Image processing with gocv
8. **Model Serving**: REST APIs, batch prediction, versioning
9. **AI APIs**: OpenAI, Claude, Hugging Face integration
10. **MLOps**: Monitoring, feature stores, A/B testing

You now understand how to integrate ML and AI into Go applications for production use!

---

**Total Word Count**: ~13,000 words  
**Status**: ✅ Complete

**Next**: Part 15 - Mobile & Cross-Platform Development
