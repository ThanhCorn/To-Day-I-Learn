# To-Day-I-Learned
# Go-routine
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
