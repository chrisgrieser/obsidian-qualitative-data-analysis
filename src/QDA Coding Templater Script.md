<%*
// #templater
//----------------------------

// Configuration
// -----------------------

// set delimiter Icon for meta-information at the end of a block
const delimiter = "♦︎";

// the search scope is restricted to notes beginning with the string set here. Leave empty to search all notes instead.
const filterString = "°";

// -----------------------

var targetNote;
if (filterString){
	targetNote = await tp.system.suggester((item) => item.path, this.app.vault.getMarkdownFiles().filter(file => file.name.startsWith(filterString)), true);
} else {
	targetNote = await tp.system.suggester((item) => item.path, this.app.vault.getMarkdownFiles(), true);
}

// save selection for highlighting
selectedText = tp.file.selection();

// select & get current Block
let cmEditorAct = this.app.workspace.activeLeaf.view.editor;
const curLine = cmEditorAct.getCursor().line;
cmEditorAct.setSelection({ line: curLine, ch: 0 }, { line: curLine, ch: 9999 });
let blockText = tp.file.selection();

//set highlight in current block, if there has been a selection
if (selectedText != ""){
	//ensures that no leading space of the selection disrupts highlighting markup
	let highlightedText = selectedText.replace (/^( ?)(.+?)( ?)$/,"$1==$2==$3");
	blockText = blockText.replace (selectedText, highlightedText);
}

//Create link to target note
let targetNoteLink = "[[" + targetNote.name.replace (/\.md$/,"") + "]]";

//function to create block-ID
function createBlockHash() {
	let result = '';
	var characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
	var charactersLength = characters.length;
	for ( var i = 0; i < 8; i++ ) {
	    result += characters.charAt(Math.floor(Math.random() * charactersLength));
	}
	return result;
}

// Check whether block already has already been referenced
// append accordingly to source note
let blockIDExists = (blockText.match(/\^\w*$/) != null)
if (blockIDExists == true){
	var id = blockText.match(/\^\w*$/)[0];
	tR = blockText.split(delimiter)[0] + delimiter + blockText.split(delimiter)[1] + targetNoteLink + " " + delimiter + " " + id + "\n";
} else {
	var id = "^" + createBlockHash();
	tR = blockText + " " + delimiter + " " + targetNoteLink + " " + delimiter + " " + `${id}`;
}

// write to target note
let blockRef = `![[${tp.file.title}#${id}]]`;
const curContent = await this.app.vault.read(targetNote);
let newContents = curContent + "**" + tp.file.title + "** " + blockRef + "\n\n";
await this.app.vault.modify(targetNote, newContents);
%>