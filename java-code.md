# A0140060A
###### \java\seedu\taskmanager\logic\commands\AddCommand.java
``` java
    public static final String MESSAGE_USAGE = COMMAND_WORD + ": Adds a task, deadline or event to the task manager. \n"
                                               + "Task Parameters: " + ItemType.TASK_WORD + " n/NAME \n"
                                               + "Deadline Parameters: " + ItemType.DEADLINE_WORD + " n/NAME ed/DATE et/TIME \n"
                                               + "Event Parameters: " + ItemType.EVENT_WORD + " n/NAME sd/DATE st/TIME ed/DATE et/TIME \n"
                                               + "Example (Task): " + COMMAND_WORD +  " " + ItemType.TASK_WORD 
                                               + " n/Win Facebook hackathon" + "\n"
                                               + "Example (Deadline): " + COMMAND_WORD +  " " + ItemType.DEADLINE_WORD 
                                               + " n/Cheat death ed/2000-12-13 et/12:34" + "\n"
                                               + "Example (Event): " + COMMAND_WORD +  " " + ItemType.EVENT_WORD
                                               + " n/Win at Life sd/1900-01-01 st/00:07 ed/2300-01-01 et/12:34 \n"
                                               + "Note: " + COMMAND_WORD + " can be replaced by " + SHORT_COMMAND_WORD + "\n"
                                               + "Note: n/ prefix for name is optional.";

    public static final String MESSAGE_SUCCESS = "Added %1$s";

    private final Item toAdd;

    /**
     * Convenience constructor for tasks. Calls primary constructor with empty fields for startDate, startTime, endDate, endTime
     *
     * @throws IllegalValueException if any of the raw values are invalid
     */
    public AddCommand(String itemType, String name, List<String> tags) throws IllegalValueException {
    	this(itemType, name, ItemDate.EMPTY_DATE, ItemTime.EMPTY_TIME, ItemDate.EMPTY_DATE, ItemTime.EMPTY_TIME, tags);
    }
    
    /**
     * Convenience constructor for deadlines. Calls primary constructor with empty fields for startDate and startTime
     *
     * @throws IllegalValueException if any of the raw values are invalid
     */
    public AddCommand(String itemType, String name, String endDate, String endTime, List<String> tagsToAdd) throws IllegalValueException {
    	this(itemType, name, ItemDate.EMPTY_DATE, ItemTime.EMPTY_TIME, endDate, endTime, tagsToAdd);
    }
    
```
###### \java\seedu\taskmanager\logic\commands\ClearCommand.java
``` java
    public static final String SHORT_COMMAND_WORD = "cl";
```
###### \java\seedu\taskmanager\logic\commands\Command.java
``` java
    public static final String MESSAGE_END_DATE_TIME_BEFORE_START_DATE_TIME = "Start datetime must come before end datetime";
    public static final String MESSAGE_DUPLICATE_ITEM = "This item already exists in the task manager";
    
    /**
     * Constructs a feedback message to summarise an operation that displayed a listing of items.
     *
     * @param displaySize used to generate summary
     * @return summary message for items displayed
     */
    public static String getMessageForItemListShownSummary(int displaySize) {
        return String.format(Messages.MESSAGE_ITEMS_LISTED_OVERVIEW, displaySize);
    }
```
###### \java\seedu\taskmanager\logic\commands\DeleteCommand.java
``` java
    public static final String SHORT_COMMAND_WORD = "del";
    
```
###### \java\seedu\taskmanager\logic\commands\DoneCommand.java
``` java
    public static final String SHORT_COMMAND_WORD = "do";
    public static final String ALTERNATE_SHORT_COMMAND_WORD = "d";
    
```
###### \java\seedu\taskmanager\logic\commands\EditCommand.java
``` java

/**
 * Edits an item identified using it's last displayed index from the task manager.
 */
public class EditCommand extends Command {
    private static final Logger logger = LogsCenter.getLogger(EditCommand.class);
    public static final String COMMAND_WORD = "edit";
    public static final String SHORT_COMMAND_WORD = "e";

    public static final String MESSAGE_USAGE = COMMAND_WORD
                                               + ": Edits the item identified by the index number" 
                                               + " used in the last item listing."
                                               + "\n" + "Parameters: INDEX (positive integer)" 
                                               + " n/[NAME]"
                                               + " sdt/[START_DATE_TIME]"
                                               + " sd/[START_DATE]"
                                               + " st/[START_TIME]"
                                               + "\n" + "                     " + "edt/[END_DATE_TIME]"
                                               + " ed/[END_DATE]"
                                               + " et/[END_TIME]"
                                               + "\n" + "At least one parameter must be specifed"
                                               + " (sdt/edt favoured over sd/st/ed/et)"
                                               + "\n" + "Example: " + COMMAND_WORD + " 1" + " n/buy milk";
    
    public static final String MESSAGE_EDIT_ITEM_SUCCESS = "Edited %1$s";
    public static final String MESSAGE_NOTHING_EDITED = "Nothing was changed!";
    
    private int targetIndex;
    private Name name;
    private ItemDate startDate;
    private ItemTime startTime;
    private ItemDate endDate;
    private ItemTime endTime;
    private UniqueTagList tagsToAdd;
    private UniqueTagList tagsToRemove;

    /*
     * Edits deadline, task, or event by index. 
     * Assumes at least one Optional parameter has contains values (user input)
     * 
     * @throws IllegalValueException if any of the raw values are invalid
     */
    public EditCommand(int targetIndex, Optional<String> name, Optional<String> startDate, 
                       Optional<String> startTime, Optional<String> endDate, Optional<String> endTime, 
                       Optional<List<String>> tagsToAdd, Optional<List<String>> tagsToRemove)
            throws IllegalValueException {
        
        assert containsInputForAtLeastOneParameter(name, startDate, startTime, endDate, 
                                                   endTime, tagsToAdd, tagsToRemove);
        
        this.targetIndex = targetIndex;
        if (name.isPresent()) {
            this.name = new Name(name.get());
        }
        if (startDate.isPresent()) {
            this.startDate = new ItemDate(startDate.get());
        }
        if (startTime.isPresent()) {
            this.startTime = new ItemTime(startTime.get());
        }
        if (endDate.isPresent()) {
            this.endDate = new ItemDate(endDate.get());
        }
        if (endTime.isPresent()) {
            this.endTime = new ItemTime(endTime.get());
        }
        
        if (tagsToAdd.isPresent()) {
            this.tagsToAdd = createUniqueTagList(tagsToAdd.get());
        }
        
        if (tagsToRemove.isPresent()) {
            this.tagsToRemove = createUniqueTagList(tagsToRemove.get());
        }
        
        logger.fine("EditCommand object successfully created!");
    }

    @Override
    public CommandResult execute() {

        UnmodifiableObservableList<ReadOnlyItem> lastShownList = model.getFilteredItemList();

        if (lastShownList.size() < targetIndex) {
            indicateAttemptToExecuteIncorrectCommand();
            return new CommandResult(Messages.MESSAGE_INVALID_ITEM_DISPLAYED_INDEX);
        }

        ReadOnlyItem itemToEdit = lastShownList.get(targetIndex - 1);
        Item itemToReplace = new Item(itemToEdit);
        
        if (isAttemptToConvertTaskToDeadline(itemToReplace)) {
            try {
                itemToReplace.convertTaskToDeadline(endDate, endTime);
            } catch (UnableToConvertFromTaskToDeadlineException utcfttde) {
                indicateAttemptToExecuteIncorrectCommand();
                return new CommandResult(utcfttde.getMessage());
            }
        }
        
        if (isInvalidInputForItemType(itemToReplace)) {
            indicateAttemptToExecuteIncorrectCommand();
            return new CommandResult(String.format(MESSAGE_INVALID_COMMAND_FORMAT, MESSAGE_USAGE));
        }
        
        setNameIfAvailable(itemToReplace);
        setStartDateIfAvailable(itemToReplace);
        setStartTimeIfAvailable(itemToReplace);
        if (!isAttemptToConvertTaskToDeadline(itemToEdit)) {
            setEndDateIfAvailable(itemToReplace);
            setEndTimeIfAvailable(itemToReplace);
        }
        addTagsIfAvailable(itemToEdit, itemToReplace);
        
        try {
            removeTagsIfApplicable(itemToEdit, itemToReplace);
        } catch (TagNotFoundException tnfe) {
            indicateAttemptToExecuteIncorrectCommand();
            return new CommandResult(String.format(MESSAGE_INVALID_COMMAND_FORMAT, tnfe.getMessage()));
        }

        if (isEventEndDateTimeBeforeStartDateTime(itemToReplace)) {
            logger.fine("detected event end datetime before start datetime");
            indicateAttemptToExecuteIncorrectCommand();
            return new CommandResult(String.format(MESSAGE_INVALID_COMMAND_FORMAT, 
                                                   MESSAGE_END_DATE_TIME_BEFORE_START_DATE_TIME));
        }
        
        try {
            model.replaceItem(itemToEdit, itemToReplace, 
                              String.format(MESSAGE_EDIT_ITEM_SUCCESS, itemToReplace));
        } catch (ItemNotFoundException pnfe) {
            assert false : "The target item cannot be missing";
        } catch (UniqueItemList.DuplicateItemException e) {
            indicateAttemptToExecuteIncorrectCommand();
            return new CommandResult(MESSAGE_NOTHING_EDITED);
        }
        return new CommandResult(String.format(MESSAGE_EDIT_ITEM_SUCCESS, itemToReplace));
    }

    
    /**
     * Checks if user is attempting to convert a task to a deadline by adding an end date 
     * or time to task
     */
    private boolean isAttemptToConvertTaskToDeadline(ReadOnlyItem item) {
        return item.getItemType().isTask() 
               && (isToBeEdited(endDate) || isToBeEdited(endTime));
    }

    /**
     * Creates a UniqueTagList from a List of Strings
     * Assumes tags is to be edited
     */
    private UniqueTagList createUniqueTagList(List<String> tags) throws IllegalValueException {
        assert isToBeEdited(tags);
        
        UniqueTagList uniqueTagList = new UniqueTagList();
        for (String tag : tags) {
            try {
                uniqueTagList.add(new Tag(tag));
            } catch (DuplicateTagException dte) {
            }
        }
        return uniqueTagList;
    }
    
    /**
     * Checks if itemToReplace is an event and if its end datetime comes before its start datetime 
     */
    private boolean isEventEndDateTimeBeforeStartDateTime(Item itemToReplace) {
        return itemToReplace.getItemType().isEvent() 
               && isEndDateTimeBeforeStartDateTime(itemToReplace.getStartDate(), itemToReplace.getStartTime(),
                                                   itemToReplace.getEndDate(), itemToReplace.getEndTime());
    }

    /**
     * Removes tags from item being edited if possible 
     */
    private void removeTagsIfApplicable(ReadOnlyItem itemToEdit, Item itemToReplace) 
                     throws TagNotFoundException{
        if (isToBeEdited(this.tagsToRemove)) {
            UniqueTagList tagListToEdit = getTagListToEditForTagRemoval(itemToEdit, itemToReplace);
            tagListToEdit.remove(tagsToRemove);
            itemToReplace.setTags(tagListToEdit);
        } 
    }

    /**
     * Gets appropriate tag list to remove tags from 
     * (dependent on whether user is adding tags in same command)
     */
    private UniqueTagList getTagListToEditForTagRemoval(ReadOnlyItem itemToEdit, Item itemToReplace) {
        if (isToBeEdited(this.tagsToAdd)) {
            return itemToReplace.getTags();
        } else {
            return itemToEdit.getTags();
        }
    }

    /**
     * Adds tags to item being edited if possible 
     */
    private void addTagsIfAvailable(ReadOnlyItem itemToEdit, Item itemToReplace) {
        if (isToBeEdited(this.tagsToAdd)) {
            UniqueTagList tagListToEdit = itemToEdit.getTags();
            tagListToEdit.mergeFrom(this.tagsToAdd);
            itemToReplace.setTags(tagListToEdit);
        }
    }

    private void setEndTimeIfAvailable(Item itemToReplace) {
        if (isToBeEdited(this.endTime)) {
            itemToReplace.setEndTime(this.endTime);
        }
    }

    private void setEndDateIfAvailable(Item itemToReplace) {
        if (isToBeEdited(this.endDate)) {
            itemToReplace.setEndDate(this.endDate);
        }
    }

    private void setStartTimeIfAvailable(Item itemToReplace) {
        if (isToBeEdited(this.startTime)) {
            itemToReplace.setStartTime(this.startTime);
        }
    }

    private void setStartDateIfAvailable(Item itemToReplace) {
        if (isToBeEdited(this.startDate)) {
            itemToReplace.setStartDate(this.startDate);
        }
    }

    private void setNameIfAvailable(Item itemToReplace) {
        if (isToBeEdited(this.name)) {
            itemToReplace.setName(this.name);
        }
    }

    /**
     * Checks if end datetime comes before start datetime
     */
    private boolean isEndDateTimeBeforeStartDateTime(ItemDate startItemDate, ItemTime startItemTime, 
                                                     ItemDate endItemDate, ItemTime endItemTime) {
        if (isEndDateEqualsStartDate(startItemDate, endItemDate)) {
            return isEndTimeBeforeStartTime(startItemTime, endItemTime);
        } else {
            return isEndDateBeforeStartDate(startItemDate, endItemDate);
        }
    }
    
    /**
     * Checks if endItemDate comes before startItemDate, false otherwise
     */
    private boolean isEndDateBeforeStartDate(ItemDate startItemDate, ItemDate endItemDate) {
        return compareStartDateToEndDate(startItemDate, endItemDate) < 0;
    }
    
    /**
     * Checks if endItemDate is equal to startItemDate, false otherwise
     */
    private boolean isEndDateEqualsStartDate(ItemDate startItemDate, ItemDate endItemDate) {
        return compareStartDateToEndDate(startItemDate, endItemDate) == 0;
    }
    
    /**
     * Compares start date to end date
     * @return -1 if endItemDate comes before startItemDate, 
     *         0 if endItemDate equals startItemDate, 1 otherwise
     */
    private int compareStartDateToEndDate(ItemDate startItemDate, ItemDate endItemDate) {
        int result = 1;
        
        try {
            SimpleDateFormat sdf = new SimpleDateFormat(ItemDate.DATE_FORMAT);
            Date startDate = sdf.parse(startItemDate.toString());
            Date endDate = sdf.parse(endItemDate.toString());
            if (endDate.before(startDate)) {
                result = -1;
            } else if (endDate.equals(startDate)) {
                result = 0;
            }
        } catch (ParseException pe) {
            assert false : "Given date(s) is/are not parsable by SimpleDateFormat";
        }
        
        return result;
    }
    
    
    /**
     * Checks if endItemTime comes before startItemTime, false otherwise
     */
    private boolean isEndTimeBeforeStartTime(ItemTime startItemTime, ItemTime endItemTime) {
        boolean result = true;
        
        try {
            SimpleDateFormat sdf = new SimpleDateFormat(ItemTime.TIME_FORMAT);
            Date startTime= sdf.parse(startItemTime.toString());
            Date endTime = sdf.parse(endItemTime.toString());
            result = endTime.before(startTime);
        } catch (ParseException pe) {
            assert false : "Given time(s) is not parsable by SimpleDateFormat";
        }
        
        return result;
    }

    /**
     * Detects if parameters input by user is not valid for the item being edited
     * Assumes ItemType consists only of Task, Deadline and Event 
     */
    private boolean isInvalidInputForItemType(ReadOnlyItem itemToReplace) { 
        ItemType itemType = itemToReplace.getItemType();
        assert itemType.isTask() || itemType.isDeadline() || itemType.isEvent();
        
        boolean result = true;
        
        if (itemType.isTask() || itemType.isDeadline()) {
            result = isToBeEdited(startDate) || isToBeEdited(startTime);
        }
        if (itemType.isEvent()) {
            result = false;
        }
        
        return result;
    }

    /**
     * Checks if at least one parameter contains input
     */
    private boolean containsInputForAtLeastOneParameter(Optional<String> name, Optional<String> startDate,
                        Optional<String> startTime, Optional<String> endDate, Optional<String> endTime, 
                        Optional<List<String>> tagsToAdd, Optional<List<String>> tagsToRemove) {
        
        return name.isPresent() || startDate.isPresent() || startTime.isPresent() || endDate.isPresent() 
               || endTime.isPresent() || tagsToAdd.isPresent() || tagsToRemove.isPresent();
    }

    /**
     * Checks if argument is to be edited
     */
    private boolean isToBeEdited(Object argument) {
        return argument != null;
    }
    
}
```
###### \java\seedu\taskmanager\logic\commands\FindCommand.java
``` java

/**
 * Finds and lists all persons in task manager whose name contains all of the argument keywords 
 * Keyword matching is not case sensitive.
 * Search results take into account typos from the user (fuzzy search) where search query can be 2 characters away from actual item name
 */
public class FindCommand extends Command {

    public static final String COMMAND_WORD = "find";
    
   
    public static final String SHORT_COMMAND_WORD = "f";

    public static final String MESSAGE_USAGE = COMMAND_WORD + ": Finds all items whose names contain any of "
                                               + "the specified keywords (case-sensitive) and displays them as a list with index numbers.\n"
                                               + "Parameters: KEYWORD [MORE_KEYWORDS]...\n"
                                               + "Example: " + COMMAND_WORD + " CS2103";

```
###### \java\seedu\taskmanager\logic\commands\HelpCommand.java
``` java
    public static final String SHORT_COMMAND_WORD = "h";
```
###### \java\seedu\taskmanager\logic\commands\ListCommand.java
``` java
    public static final String SHORT_COMMAND_WORD = "l";
```
###### \java\seedu\taskmanager\logic\commands\ListDeadlineCommand.java
``` java
    public static final String SHORT_COMMAND_WORD = "ld";
```
###### \java\seedu\taskmanager\logic\commands\ListEventCommand.java
``` java
    public static final String SHORT_COMMAND_WORD = "le";
```
###### \java\seedu\taskmanager\logic\commands\ListNotDoneCommand.java
``` java
/**
 * Lists all not done (uncompleted) items in the task manager to the user.
 */
public class ListNotDoneCommand extends Command {

    public static final String COMMAND_WORD = "listnotdone";
    
    public static final String SHORT_COMMAND_WORD = "lnd";
    
    public static final String MESSAGE_SUCCESS = "Listed all uncompleted items";

    public ListNotDoneCommand() {}

    @Override
    public CommandResult execute() {
        model.updateFilteredListToShowNotDone();
        return new CommandResult(MESSAGE_SUCCESS);
    }
}
```
###### \java\seedu\taskmanager\logic\commands\ListTaskCommand.java
``` java
    public static final String SHORT_COMMAND_WORD = "lt";
```
###### \java\seedu\taskmanager\logic\commands\NotDoneCommand.java
``` java
    public static final String SHORT_COMMAND_WORD = "nd";
    
```
###### \java\seedu\taskmanager\logic\commands\SelectCommand.java
``` java
    public static final String SHORT_COMMAND_WORD = "s";
    
    public static final String MESSAGE_USAGE = COMMAND_WORD
            + ": Selects the item identified by the index number used in the last item listing.\n"
            + "Parameters: INDEX (must be a positive integer)\n"
            + "Example: " + COMMAND_WORD + " 1";

    public static final String MESSAGE_SELECT_ITEM_SUCCESS = "Selected %1$s";
```
###### \java\seedu\taskmanager\logic\commands\UndoCommand.java
``` java
    public static final String SHORT_COMMAND_WORD = "u";
```
###### \java\seedu\taskmanager\logic\parser\Parser.java
``` java
    /*
     * Used to separate parameters from their delimiters
     */
    private static final Prefix namePrefix = new Prefix("n/");
    private static final Prefix startDatePrefix = new Prefix("sd/");
    private static final Prefix startTimePrefix = new Prefix("st/");
    private static final Prefix startDateTimePrefix = new Prefix("sdt/");
    private static final Prefix endDatePrefix = new Prefix("ed/");
    private static final Prefix endTimePrefix = new Prefix("et/");
    private static final Prefix endDateTimePrefix = new Prefix("edt/");
    private static final Prefix tagPrefix = new Prefix("#");
    private static final String removeTagPrefixString = "-";
    
    private static final int PARSE_DATETIME_ARRAY_DATE_INDEX = 0;
    private static final int PARSE_DATETIME_ARRAY_TIME_INDEX = 1;
    
```
###### \java\seedu\taskmanager\logic\parser\Parser.java
``` java
    /**
     * Parses arguments in the context of the edit item command.
     * Assumes args is not null
     *
     * @param args full command args string
     * @return the prepared EditCommand
     */
    private Command prepareEdit(String args) {
        assert args != null;
        final Matcher matcher = EDIT_COMMAND_ARGS_FORMAT.matcher(args.trim());
        
        if (matcher.matches() && hasParsableIndex(matcher.group("targetIndex"))) {
            Integer index = parseIndex(matcher.group("targetIndex")).get();
            
            String editCommandArgs = matcher.group("editCommandArguments");
            ArgumentTokenizer argsTokenizer = new ArgumentTokenizer(namePrefix, startDatePrefix, 
                                                    startTimePrefix, startDateTimePrefix, endDatePrefix, 
                                                    endTimePrefix, endDateTimePrefix, tagPrefix);
            logger.fine("In prepareEdit, before tokenize");
            argsTokenizer.tokenize(editCommandArgs);
            
            //Capture argument values into their respective variables if available
            Optional<String> name = getParsedArgumentFromArgumentTokenizer(argsTokenizer, namePrefix);
            Optional<String> startDate = getParsedArgumentFromArgumentTokenizer(argsTokenizer, startDatePrefix);
            Optional<String> startTime = getParsedArgumentFromArgumentTokenizer(argsTokenizer, startTimePrefix);
            Optional<String> endDate = getParsedArgumentFromArgumentTokenizer(argsTokenizer, endDatePrefix);
            Optional<String> endTime = getParsedArgumentFromArgumentTokenizer(argsTokenizer, endTimePrefix);
            Optional<String> startDateTime = getParsedArgumentFromArgumentTokenizer(argsTokenizer, startDateTimePrefix);
            Optional<String> endDateTime = getParsedArgumentFromArgumentTokenizer(argsTokenizer, endDateTimePrefix);
            Optional<List<String>> tagsToAdd = getParsedTagsToAddFromArgumentTokenizer(argsTokenizer, tagPrefix);
            Optional<List<String>> tagsToRemove = getParsedTagsToRemoveFromArgumentTokenizer(argsTokenizer, tagPrefix);
            
            try {
                if (startDateTime.isPresent()) {
                    String[] startDateTimeArr = parseDateTime(startDateTime.get(), ItemDate.DATE_FORMAT, 
                                                              ItemTime.TIME_FORMAT);
                    
                    startDate = Optional.of(startDateTimeArr[PARSE_DATETIME_ARRAY_DATE_INDEX]);
                    startTime = Optional.of(startDateTimeArr[PARSE_DATETIME_ARRAY_TIME_INDEX]);
                }
                
                if (endDateTime.isPresent()) {
                    String[] endDateTimeArr = parseDateTime(endDateTime.get(), ItemDate.DATE_FORMAT, 
                                                            ItemTime.TIME_FORMAT);
                    
                    endDate = Optional.of(endDateTimeArr[PARSE_DATETIME_ARRAY_DATE_INDEX]);
                    endTime = Optional.of(endDateTimeArr[PARSE_DATETIME_ARRAY_TIME_INDEX]);
                }
                
                if (containsInputForAtLeastOneParameter(name, startDate, startTime, endDate, 
                                                        endTime, tagsToAdd, tagsToRemove)) {
                    return new EditCommand(index, name, startDate, startTime, 
                                           endDate, endTime, tagsToAdd, tagsToRemove);
                }
            } catch (IllegalValueException ive) {
                return new IncorrectCommand(ive.getMessage());
            }   
        }
        return new IncorrectCommand(String.format(MESSAGE_INVALID_COMMAND_FORMAT, EditCommand.MESSAGE_USAGE));
    }

    /**
     * Checks if index can be parsed
     * @param indexToParse String containing user input for index parameter
     * @return true if index can be successfully parsed
     */
    private boolean hasParsableIndex(final String indexToParse) {
        return parseIndex(indexToParse).isPresent();
    }

    /**
     * Checks if at least one parameter contains input
     */
    private boolean containsInputForAtLeastOneParameter(Optional<String> name, Optional<String> startDate,
                        Optional<String> startTime, Optional<String> endDate, Optional<String> endTime, 
                        Optional<List<String>> tagsToAdd, Optional<List<String>> tagsToRemove) {
        
        return name.isPresent() || startDate.isPresent() || startTime.isPresent() || endDate.isPresent() 
               || endTime.isPresent() || tagsToAdd.isPresent() || tagsToRemove.isPresent();
    }

    

    /**
     * Gets the parsed argument if available 
     */
    private Optional<String> getParsedArgumentFromArgumentTokenizer(ArgumentTokenizer argsTokenizer, Prefix argumentPrefix) {
            return argsTokenizer.getValue(argumentPrefix); 
    }

    /**
     * Gets the list of parsed tags to be removed if available 
     */
    private Optional<List<String>> getParsedTagsToRemoveFromArgumentTokenizer(ArgumentTokenizer argsTokenizer, Prefix tagPrefix) {
        Optional<List<String>> tags = argsTokenizer.getAllValues(tagPrefix);
        
        if (!tags.isPresent()) {
            return Optional.empty();
        }
        
        logger.fine("Before remove tags check");
        
        List<String> tagsToRemove = new ArrayList<String>();
        for (String tag : tags.get()) {
            if (tag.length() > 0 && isATagToBeRemoved(tag)) {                            
                tagsToRemove.add(extractTagToBeRemoved(tag));
            }
        }
        if (!tagsToRemove.isEmpty()) {
            return Optional.of(tagsToRemove);    
        } else {
            return Optional.empty();
        }
    }
    
    /**
     * Gets the list of parsed tags to be added if available 
     */
    private Optional<List<String>> getParsedTagsToAddFromArgumentTokenizer(ArgumentTokenizer argsTokenizer, Prefix tagPrefix) {
        Optional<List<String>> tags = argsTokenizer.getAllValues(tagPrefix);
        
        if (!tags.isPresent()) {
            return Optional.empty();
        }
        
        List<String> tagsToAdd = new ArrayList<String>();
        for (String tag : tags.get()) {
            if (tag.length() > 0 && !isATagToBeRemoved(tag)) {                            
                tagsToAdd.add(tag);
            }
        }
        if (!tagsToAdd.isEmpty()) {
            return Optional.of(tagsToAdd);    
        } else {
            return Optional.empty();
        }
    }

    /**
     * Extracts tag from string containing tag and tag removal prefix
     * Assumes tag string contains tag removal prefix
     */
    private String extractTagToBeRemoved(String tag) {
        assert isATagToBeRemoved(tag);
        logger.fine("In processTagToBeRemoved, before return");
        return tag.substring(removeTagPrefixString.length(), tag.length());
    }

    /**
     * Checks if tag is to be removed from the item's current tag list.
     */
    private boolean isATagToBeRemoved(String tag) {
        return tag.substring(0, removeTagPrefixString.length()).equals(removeTagPrefixString);
    }

    /**
     * Parses date and time from argument acquired through NLP input
     *  Assumes datetime contains user input and time and date formats are not null
     *  
     * @param datetime datetime to be parsed
     * @param dateFormat the format the date should be returned in
     * @param timeFormat the format the time should be returned in
     * @return parsed argument as string or null if argument not parsed 
     */
    private String[] parseDateTime(String datetime, String dateFormat, String timeFormat) throws IllegalValueException {
        assert dateFormat != null && !dateFormat.isEmpty();
        assert timeFormat != null && !timeFormat.isEmpty();
        assert containsInput(datetime);
        
        String[] parsedDateTime = new String[2];
        
        List<Date> dateTimes = new PrettyTimeParser().parse(datetime);
        
        if (dateTimes.isEmpty()) {
            throw new IllegalValueException(MESSAGE_DATETIME_PARSE_FAILURE);
        }
        
        Date prettyParsedDateTime = dateTimes.get(0);
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat(dateFormat);
        SimpleDateFormat simpleTimeFormat = new SimpleDateFormat(timeFormat);
        String parsedDate = simpleDateFormat.format(prettyParsedDateTime);
        String parsedTime = simpleTimeFormat.format(prettyParsedDateTime);
        
        
        parsedDateTime[PARSE_DATETIME_ARRAY_DATE_INDEX] = parsedDate;
        parsedDateTime[PARSE_DATETIME_ARRAY_TIME_INDEX] = parsedTime;
        
        return parsedDateTime;
    }
    
```
###### \java\seedu\taskmanager\model\item\Item.java
``` java
    public void setName(Name name) {
        this.name = name;
    }
    
    public void setStartDate(ItemDate startDate) {
        this.startDate = startDate;
    } 
    
    public void setStartTime(ItemTime startTime) {
        this.startTime = startTime;
    }
    
    public void setEndDate(ItemDate endDate) {
        this.endDate = endDate;
    }
    
    public void setEndTime(ItemTime endTime) {
        this.endTime = endTime;
    }
    
    public void convertTaskToDeadline(ItemDate endDate, ItemTime endTime) throws UnableToConvertFromTaskToDeadlineException {
        if (!CollectionUtil.isAnyNull(endDate, endTime)) {
            convertItemTypeFromTaskToDeadline();
            setEndDate(endDate);
            setEndTime(endTime);
        } else {
            throw new UnableToConvertFromTaskToDeadlineException();
        }
    }
    
    /*
     * Converts this item from a task into a deadline;
     */
    private void convertItemTypeFromTaskToDeadline() {
        assert itemType.isTask();
        try {
            this.itemType = new ItemType(ItemType.DEADLINE_WORD);
        } catch (IllegalValueException e) {
            assert false : "ItemType.DEADLINE_WORD constant is now an illegal value";
        }
    }
    
```
###### \java\seedu\taskmanager\model\item\ItemDate.java
``` java
    public static final String EMPTY_DATE = "";


    public final String value;
    
    /**
     * Convenience constructor for empty ItemDate
     */
    public ItemDate() {
        value = EMPTY_DATE;
    }
    
```
###### \java\seedu\taskmanager\model\item\ItemTime.java
``` java
    public static final String EMPTY_TIME = "";

    public final String value;

    /**
     * Convenience constructor for empty ItemTime
     */
    public ItemTime() {
        value = EMPTY_TIME;
    }
```
###### \java\seedu\taskmanager\model\item\ItemType.java
``` java
    /**
     * Returns true if item type is 'task'.
     */
    public boolean isTask() {
        return value.equals(TASK_WORD);
    }
    
    /**
     * Returns true if item type is 'event'.
     */
    public boolean isEvent() {
        return value.equals(EVENT_WORD);
    }
    
    /**
     * Returns true if item type is 'deadline'.
     */
    public boolean isDeadline() {
        return value.equals(DEADLINE_WORD);
    }
    
```
###### \java\seedu\taskmanager\model\item\ReadOnlyItem.java
``` java
    /**
     * Formats the person as text, showing all contact details.
     */
    default String getAsText() {
        final StringBuilder builder = new StringBuilder();
        builder.append(getItemType())
               .append("\n")
               .append("Name: ")
               .append(getName());
        if (getItemType().toString().equals(ItemType.EVENT_WORD)) {
            builder.append("\n")
                   .append("Start Date: ")
                   .append(getStartDate())
                   .append(" Start time: ")
                   .append(getStartTime());
        }
        if (getItemType().toString().equals(ItemType.DEADLINE_WORD) || getItemType().toString().equals(ItemType.EVENT_WORD)) {
            builder.append("\n")
                   .append("End Date: ")
                   .append(getEndDate())
                   .append(" End time: ")
                   .append(getEndTime());
        }
        builder.append("\n").append("Tags: ");
        getTags().forEach(builder::append);
        return builder.toString();
    }
```
###### \java\seedu\taskmanager\model\item\UniqueItemList.java
``` java
    /**
     * Replaces an item in the list.
     *     
     * @throws ItemNotFoundException if no such item could be found in the list.
     * @throws DuplicateItemException if the replacing item is a duplicate of an existing item in the list.
     */
    public void replace(ReadOnlyItem target, Item toReplace) throws ItemNotFoundException, DuplicateItemException {
        assert target != null;
        assert toReplace != null;
        final int itemIndex = internalList.indexOf(target);
        if (itemIndex == -1) {
            throw new ItemNotFoundException();
        }
        if (contains(toReplace)) {
            throw new DuplicateItemException();
        }
        internalList.set(itemIndex, toReplace);
    }
    
```
###### \java\seedu\taskmanager\model\Model.java
``` java
    /** Replaces the given item */
    void replaceItem(ReadOnlyItem target, Item toReplace, String actionTaken) throws UniqueItemList.ItemNotFoundException, UniqueItemList.DuplicateItemException;
```
###### \java\seedu\taskmanager\model\Model.java
``` java
    /** Updates the filter of the filtered person list to show all not done (uncompleted) items */
    void updateFilteredListToShowNotDone();
    
```
###### \java\seedu\taskmanager\model\ModelManager.java
``` java
    @Override
    public synchronized void replaceItem(ReadOnlyItem target, Item toReplace, String actionTaken) 
            throws ItemNotFoundException, UniqueItemList.DuplicateItemException {
        taskManager.replaceItem(target, toReplace);
        updateFilteredListToShowAll();
        indicateTaskManagerChanged(actionTaken);
    }
```
###### \java\seedu\taskmanager\model\ModelManager.java
``` java
    @Override
    public void updateFilteredListToShowNotDone() {
        updateFilteredItemList(new PredicateExpression(new StatusQualifier(false)));
    }
```
###### \java\seedu\taskmanager\model\ModelManager.java
``` java
    private class StatusQualifier implements Qualifier {
        private boolean isDone;

        StatusQualifier(boolean isDone) {
            this.isDone = isDone;
        }
        
        @Override
        public boolean run(ReadOnlyItem item) {
            return item.getDone() == isDone;
        }

        @Override
        public String toString() {
            return "done=" + isDone;
        }
    }
}
```
###### \java\seedu\taskmanager\model\tag\UniqueTagList.java
``` java
    /**
     * Signals that an operation tried to access a nonexistent tag.
     */
    public static class TagNotFoundException extends IllegalValueException {
        public static final String MESSAGE_TAG_NOT_FOUND = "Tag %1$s does not exist! Tags must exist in order to be deletable";
        protected TagNotFoundException(Tag tag) {
            super(String.format(MESSAGE_TAG_NOT_FOUND, tag));
        }
    }
```
###### \java\seedu\taskmanager\model\tag\UniqueTagList.java
``` java
    /**
     * Removes a Tag from the list.
     * Assumes toRemove is not null
     * 
     * @throws TagNotFoundException if the Tag to remove does not exist in the list.
     */
    public void remove(Tag toRemove) throws TagNotFoundException {
        assert toRemove != null;
        if (contains(toRemove)) {
            this.internalList.remove(toRemove);
        } else {
            throw new TagNotFoundException(toRemove);
        }
    }
    
    /**
     * removes every tag in the argument list from this list.
     * @param tagsToRemove 
     * @throws TagNotFoundException if a Tag to remove does not exist in the list.
     */
    public void remove(UniqueTagList tagsToRemove) throws TagNotFoundException {
        for (Tag tag : tagsToRemove) {
            remove(tag);
        }
    }
```
###### \java\seedu\taskmanager\model\TaskManager.java
``` java
    /**
     * Edits an item in the task manager.
     * Also checks the new item's tags and updates {@link #tags} with any new tags found,
     * and updates the Tag objects in the item to point to those in {@link #tags}.
     *     
     * @throws ItemNotFoundException if no such item could be found in the task manager.
     * @throws UniqueItemList.DuplicateItemException if an equivalent item already exists.
     */
    public void replaceItem(ReadOnlyItem target, Item toReplace) 
            throws UniqueItemList.ItemNotFoundException, UniqueItemList.DuplicateItemException {
        syncTagsWithMasterList(toReplace);
        items.replace(target, toReplace);
    }
```
###### \resources\view\DarkTheme.css
``` css
.cell_big_label {
    -fx-text-fill: derive(black, 35%);
    -fx-font-size: 24px;
    -fx-max-width: 300;
    -fx-wrap-text: true;
}
```
###### \resources\view\DarkTheme.css
``` css
.anchor-pane-with-border {
	-fx-font-family: Palatino;
    -fx-text-fill: derive(black, 35%);
    -fx-background-color: main-bg-color;
    -fx-border-color: derive(white, 100%);
    -fx-border-width: 1px;
}

```
###### \resources\view\DarkTheme.css
``` css
.context-menu:hover {
    -fx-background-color: "#2daeff";
}

.menu-item:focused {
    -fx-background-color: "#2daeff";
    -fx-text-fill: white;
}
```
###### \resources\view\ItemListCard.fxml
``` fxml
                        <HBox alignment = "CENTER_LEFT" spacing = "5">
                            <children>
                                <HBox>
                                	
                                    <TextFlow>
                                   		<Text fx:id = "name" text = "\$first">
                                   			<font>
                								<Font name = "Palatino" size = "24.0" />
           									</font>
           								</Text>
                                    </TextFlow>
                                </HBox>
                            </children>
                        </HBox>
                        <HBox alignment = "CENTER_LEFT" spacing = "5">
	                        <TextFlow>
	                       		<Text fx:id = "itemType" text = "\$itemType" >
	                       			<font>
	    								<Font name = "Palatino" size = "16.0" />
									</font>
								</Text>
								<Text text = " "/>
								<Text fx:id = "tags" text = "\$tags" >
	                       			<font>
	    								<Font name = "Palatino" size = "16.0" />
									</font>
								</Text>
	                        </TextFlow>
                        </HBox>
```
