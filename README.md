# To-Day-I-Learned
# Channel
- When using go routine which returns multiple error:
- if we change errCh := make(chan error) to errCh := make(chan error, 1).
- Now errCh is a buffer channel,if new errors are produced before the previous errors are consumed, the oldest error will be overwritten
- Best Practice channel Size is One or None

func worker(id int, wg *sync.WaitGroup, errCh chan<- error) {
	defer wg.Done()

	// Simulate work
	if id%2 == 0 {
		errCh <- errors.New(fmt.Sprintf("error from worker %d", id))
	}
}

func main() {
	var wg sync.WaitGroup
	errCh := make(chan error)

	// Start workers
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go worker(i, &wg, errCh)
	}

	// Wait for all workers to finish
	go func() {
		wg.Wait()
		close(errCh)
	}()

	// Collect errors
	var errors []error
	for err := range errCh {
		errors = append(errors, err)
	}

	// Process errors
	for _, err := range errors {
		fmt.Println("Error:", err)
	}
}

# Go routine 
- wg.Add(len(arrays)) or wg.Add(1) inside the for loop is the same
- even though you have the condition to return early in the for loop ( if i >50 {return} ).The go routine is always run 100 times and it will not catch the deadlocks. Because we call 
defer wg.Done() first line in go routine and it will be decrement by 1 the waitgroup and we go routine have to wait for all the goroutine finish. Simply if any I < 50 it will execute done soon

func main() {
	var wg sync.WaitGroup
	done := make(chan struct{}) // Channel to signal that all goroutines have finished
	for i := 0; i < 100; i++ {
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			fmt.Println("Processing goroutine", i)
			if i >= 50 {
				return 
			}
		}(i)
	}
	// Wait for all goroutines to finish
	go func() {
		wg.Wait()
		close(done) // Close the channel to indicate that all goroutines have finished
	}()
	// Wait for the done channel to be closed
	<-done
	fmt.Println("All goroutines completed.")
}


![image](https://github.com/ThanhCorn/To-Day-I-Learned/assets/99862284/08ad06c3-4436-4352-9d86-7397f5be2a64)

