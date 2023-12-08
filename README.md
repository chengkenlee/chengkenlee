### Hello，World 👋
<div>
	<img align="left" src="https://github-readme-stats.vercel.app/api/top-langs/?username=chengkenlee&amp;layout=compact" /> <img align="right" src="https://github-readme-stats.vercel.app/api?username=chengkenlee&show_icons=true&icon_color=CE1D2D&text_color=718096&bg_color=ffffff&hide_title=true" /> 
</div>
<div><a href="https://www.credly.com/badges/71675ab2-4807-450c-81e6-1ecccb71a7b1/public_url">
<img src="https://github.com/chengkenlee/chengkenlee/assets/53367668/ad67ffde-44a6-4802-b2b7-6981c6e34dd6" alt="chengken"></a>
</div>

```
func runShell(prog, fileName string, timeOut int) error {
	var (
		cmd *exec.Cmd
		f   *os.File
		err error
	)
	if timeOut == 0 {
		cmd = exec.Command("bash", "-c", prog)
	} else {
		ctx, cancelFunc := context.WithTimeout(context.Background(), time.Duration(timeOut)*time.Second)
		defer cancelFunc()
		cmd = exec.CommandContext(ctx, "bash", "-c", prog)
	}

	if f, err = os.OpenFile(fileName, os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0600); err != nil {
		return err
	}
	defer func() {
		_ = f.Close()
	}()
	_, _ = io.WriteString(f, fmt.Sprintf("Run: %s\nOutput_____________\n", prog))

	stdoutIn, _ := cmd.StdoutPipe()
	stderrIn, _ := cmd.StderrPipe()
	var errStdout, errStderr error
	stdout := io.MultiWriter(f, os.Stdout)
	stderr := io.MultiWriter(f, os.Stderr)
	err = cmd.Start()
	if err != nil {
		errMsg := fmt.Sprintf("cmd.Start() failed with %s ", err)
		_, _ = io.WriteString(stdout, errMsg+"\n")
		return fmt.Errorf(errMsg)
	}
	go func() {
		_, errStdout = io.Copy(stdout, stdoutIn)
	}()
	go func() {
		_, errStderr = io.Copy(stderr, stderrIn)
	}()
	err = cmd.Wait()
	if err != nil {
		errMsg := fmt.Sprintf("cmd.Run() failed with %s ", err)
		_, _ = io.WriteString(stderr, errMsg+"\n")
		return fmt.Errorf("%v", errMsg)
	}
	if errStdout != nil || errStderr != nil {
		errMsg := fmt.Sprintf("failed to capture stdout or stderr, errStdout:%s errStderr:%s", errStdout, errStderr)
		_, _ = io.WriteString(stderr, errMsg+"\n")
		return fmt.Errorf("%v", errMsg)
	}
	return nil
}

```
