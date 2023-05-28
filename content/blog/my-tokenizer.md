---
title: 'So, I have built a bare-bones tokenizer'
date: 2023-05-21
Tags: [English]
Categories: [article]
draft: true
---


There are two main use cases for the tokenizer that influence how multiprocessing can be applied: 
- When directly tokenizing a large string of text.
- When tokenizing an array of shorter texts, usually sentences.

The second case is trivial: divide the array into batches and tokenize each of them independently.
However, the first point is trickier. It could be simplified to the array case by splitting it into sentences.
But I have not implemented sentence splitting, nor do I think that you will always want to pay the processing
cost of splitting a large text into sentences. So what we can do instead is to directly split the text into 
as many chunks as multiprocessing cores we have. My implementation tries to guarantee clean slices by 
following the following steps:
- Generate **N** -1 pointers that split the text into **N** even chunks, where **N** is the number of parallel processes to launch.
- For each pointer:
  1. Look into characters in positions -1, 0 and 1 from the pointer position.
  2. Determine if a valid split of the text can be created from the pointer.
  3. If so, proceed to the next pointer. Otherwise, shift the pointer one position to the left and move to step 1.




### Moses parallelism

This Perl code is an example of parallelizing the tokenize function using threads. Let's break down how it works:

1. The code uses a while loop to read input from STDIN line by line.
2. For each line read, the variable $count_sentences is incremented by 1, and the line is added to the @batch_sentences array.
3. If the number of lines in @batch_sentences reaches or exceeds the maximum limit ($NUM_SENTENCES_PER_THREAD * $NUM_THREADS), the batch is ready for processing.
4. Inside the if condition, a for loop is used to assign work to each thread. The loop creates a new thread (Thread) for each sub-batch of sentences.
5. The @subbatch_sentences array is created by slicing the @batch_sentences array from $start_index to $end_index.
6. A new thread is created by calling the tokenize_batch function with @subbatch_sentences as arguments. The thread is stored in the @thread_list array.
7. After creating all the threads, a foreach loop iterates over each thread in @thread_list.
8. The join method is called on each thread to wait for it to finish executing. This method returns the value returned by the tokenize_batch function.
9. The resulting tokenized list from each thread is printed to STDOUT using the print statement.
10. Finally, the @thread_list and @batch_sentences arrays are reset for the next batch.
After the while loop, there is an additional block of code to handle the last batch of sentences that may not reach the maximum limit. This block is similar to the one inside the if condition and performs the same steps.

In summary, this code reads input sentences from STDIN, batches them, and distributes the batches to multiple threads for parallel processing. Each thread calls the tokenize_batch function on its assigned sub-batch of sentences. The resulting tokenized lists from each thread are then printed.

In Perl, the default threading implementation, known as "interpreter-based threads" or "ithreads," typically does not automatically distribute threads across multiple cores. Therefore, if you run this code using the default Perl interpreter, it is likely that the threads will be executed on a single core.
