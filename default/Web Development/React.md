React is actually simply a difference tool. It tells the difference between the real html and the virtual html define by [[JSX]] a new syntax which combines [[HTML]], [[CSS]], [[JS]]. By knowing the difference we can update modify the real html easily, and keep in mind that web page structure itself is kind of a tree structure, so if we know the diff we can update it conveniently. Such as git is also using a tree structure and diff to record changes.
```
function Component(prop) {
	return <div>
	<p>{prop.text}</p>
	</div>
}
```
`{}` indicates the contents inside it are JS code. `prop` is the attributes that passed down from parent.

# Bootstrap
Sizes: `xs sm md lg xl xxl`