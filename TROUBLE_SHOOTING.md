LLMs can select random elements in the inference process depending on parameter settings. Since this selection is not always correct, there is a possibility of writing problematic code even when using the same context and prompt. These issues can also be resolved in a `VIBE` way. Below are some examples of how to solve problems by interacting with LLM in various situations.

### Check the error log and forward it to the LLM.

LLM knows more than you might think. By checking and transmitting error logs that appear on the screen, you can guide LLM to recognize and resolve its own problems. The most basic method is to transmit error toast messages output from the framework to LLM, or to transmit error messages output in the browser's developer tools to LLM.

### When the decimal value of a token is too long

Enter this sentence into the prompt: `Please check whether the token amount is being displayed according to the correct decimal formatting rules.`

### The number of tokens is incorrectly displayed

Enter this sentence into the prompt: `Find the code that displays the token amount and verify whether the decimals are correctly applied.`

### Change the part expressed as WTON to TON
Enter this sentence into the prompt: `Replace all instances of WTON with TON, and ensure that the correct decimals adjustment is applied during the conversion.`

### Change button color
Enter this sentence into the prompt: `Please change the color of the Withdraw button to blue.` or `Please change the color of the Withdraw button to a different color.`

### When no solution comes to mind
Although the situation is desperate, it is still too early to give up everything. Please enter the following sentence into the prompt:
`I don't know the exact reason, but there is a problem with the code. Please check what the problem is and fix it.`