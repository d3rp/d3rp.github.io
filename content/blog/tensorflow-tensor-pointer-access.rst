---
title: "Access Pointer of Tensorflow Tensor"
date: 2019-09-17T21:14:28+03:00
author: Seyoung Park
draft: false
---

When I was porting the PyTorch front-end of `Redner <https://github.com/BachiLi/redner>`_, a differentiable computer graphics renderer, to Tensorflow, I needed to access pointers of tensors. It's because how Redner works. Redner has its rendering logic in the C++ back-end, and used PyTorch for backpropagation & training framework.

PyTorch `Tensor` has a method called `data_ptr() <https://pytorch.org/docs/stable/tensors.html#torch.Tensor.data_ptr>`_ which returns the address of the first element of itself. `When you call that method on a tensor in GPU, you get a pointer that points to GPU memory <https://github.com/pytorch/pytorch/issues/1649#issuecomment-478730931>`_. Unfortunately, Tensorflow does not have this API. I did post a question on StackOverflow but I got one downvote, and the post got deleted. I did bang my head against my wall more than a few times but eventually, I implemented Tensorflow version of `data_ptr()` (with help of my guru `Markus Kettunen <https://scholar.google.com/citations?user=Z_1Gr-0AAAAJ&hl=en&oi=ao>`_).

The following is the Tensorflow ``data_ptr`` Ops. (The GPU support was added by the Redner author `Tzu-Mao Li <https://scholar.google.com/citations?hl=en&user=Y7MCOdYAAAAJ>`_.)

{{< highlight bash >}}

    #include "tensorflow/core/framework/op.h"
    #include "tensorflow/core/framework/shape_inference.h"
    #include "tensorflow/core/framework/op_kernel.h"
    #include <stdint.h>
    #include <climits>

    using namespace tensorflow;

    /* Tensorflow custom ops does not allow parameter types of list of 
    various data types. Therefore, we can't pass a list but we have
    to pass each objects individually. 
    Consult Tensorflow source code: /tensorflow/core/framework/tensor.h
    for what is supported by Tensorflow
    */

    REGISTER_OP("DataPtr")
        .Attr("T: {float, int32} = DT_INT32")  // To preserve backwards compatibility, you should specify a default value when adding an attr to an existing op:
        .Input("input: T")  // Tensor
        .Output("output: uint64")  // scalar
        .SetShapeFn([](::tensorflow::shape_inference::InferenceContext* c) {
        c->set_output(0, {}); // scalar
        return Status::OK();
        });

    template <typename T>
    class DataPtrOp : public OpKernel {
    public:
    explicit DataPtrOp(OpKernelConstruction* context) : OpKernel(context) {}

    void Compute(OpKernelContext* context) override {
        // Grab the input tensor

        const Tensor& input_tensor = context->input(0);
        auto tensor = reinterpret_cast<const T*>(input_tensor.tensor_data().data());

        // Create an output tensor
        // NOTE: The output datatype must match the Ops definition!!!.
        Tensor* output_tensor = NULL;
        // Always allocate on CPU
        AllocatorAttributes alloc_attr;
        alloc_attr.set_on_host(true);
        OP_REQUIRES_OK(context, 
        context->allocate_output(0, {},  // Initialize a one-element scalar
        &output_tensor,
        alloc_attr)
        );
        auto output_flat = output_tensor->flat<uint64>();

        // Cast pointer to unsigned long int
        uintptr_t addr = (uintptr_t)tensor;

        // Cast unsigned long int -> unsigned int64
        uint64 addr_converted = addr;

        output_flat(0) = addr_converted;
    }
    };

    // Polymorphism: https://www.tensorflow.org/guide/extend/op#polymorphism
    REGISTER_KERNEL_BUILDER(
    Name("DataPtr")
    .Device(DEVICE_CPU)
    .TypeConstraint<int32>("T"),
    DataPtrOp<int32>);
    REGISTER_KERNEL_BUILDER(
    Name("DataPtr")
    .Device(DEVICE_CPU)
    .TypeConstraint<float>("T"),
    DataPtrOp<float>);
    REGISTER_KERNEL_BUILDER(
    Name("DataPtr")
    .Device(DEVICE_GPU)
    .TypeConstraint<int32>("T")
    .HostMemory("output"),
    DataPtrOp<int32>);
    REGISTER_KERNEL_BUILDER(
    Name("DataPtr")
    .Device(DEVICE_GPU)
    .TypeConstraint<float>("T")
    .HostMemory("output"),
    DataPtrOp<float>);

{{< /highlight >}}

 
The entire step to add a new Op is out of the scope for this blog post. You can consult the official documentation: https://www.tensorflow.org/guide/extend/op and also our `CMakeLists.txt <https://github.com/BachiLi/redner/blob/master/pyredner_tensorflow/custom_ops/CMakeLists.txt>`_. In order to see how we use this Ops, check out the `Redner repo <https://github.com/BachiLi/redner/blob/master/pyredner_tensorflow/__init__.py>`_.

Note: once you get the pointer in C++, there is no difference between ``tf.constant`` or ``tf.variable``; they are all mutable arrays. However, there is a reason why ``tf.constant`` is a constant. If you are interested in what could go wrong when you change ``tf.constant`` in C++, check out this `PR thread <https://github.com/BachiLi/redner/pull/47>`_ .
