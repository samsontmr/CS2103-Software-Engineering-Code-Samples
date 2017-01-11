# A0140060A
###### \java\guitests\EditCommandTest.java
``` java
/*
 * Integration tests for the edit command
 */
public class EditCommandTest extends TaskManagerGuiTest {
    private static final Logger logger = LogsCenter.getLogger(EditCommandTest.class);
    
    private String firstTestName = "Survive 2103";
    private String secondTestName = "Survive the semester";
    private String thirdTestName = "Survive university";
    private String validStartDate = "1900-05-05";
    private String invalidOrderStartDate = "2020-05-05";
    private String invalidOrderEndDate = "1990-05-05";
    private String invalidOrderEndTime = "01:06";
    private String invalidOrderStartTime = "22:06";
    private String validStartTime = "22:06";
    private String validEndDate = "2320-12-05";
    private String validEndTime = "23:06";
    private String validNlpStartDateTime = "10 years before 3rd june 2016 at 2pm";
    private String expectedNlpStartDate = "2006-06-03";
    private String expectedNlpStartTime = "14:00";
    private String validNlpEndDateTime = "one year from 3rd june 2016 at 2pm";
    private String expectedNlpEndDate = "2017-06-03";
    private String expectedNlpEndTime = "14:00";
    private String invalidTime = "46:99";
    private String invalidDate = "12345679";
    private String nonexistentTag = "nonexistent";
    private UniqueTagList tagsToAdd = new UniqueTagList();
    private UniqueTagList tagsToRemove = new UniqueTagList();
    TestItem[] currentList;

    @Before
    public void setUp() throws Exception {
        tagsToAdd.add(new Tag("work"));
        tagsToRemove.add(new Tag("play"));
    }
    
    @Test
    public void edit_Success() throws IllegalValueException {
        //edit the first in the list which is an event
        TestItem[] currentList = td.getTypicalItems();
        int targetIndex = 1;
        TestItem itemToEdit = currentList[targetIndex-1];
        
        assert itemToEdit.getItemType().isEvent();
        
        assertEditSuccess(COMMAND_WORD, targetIndex, firstTestName, null, null, null, null, null, null,
                          null, null, currentList);
        
        assertEditSuccess(SHORT_COMMAND_WORD, targetIndex, null, validStartDate, null, null, null, null, null, 
                          null, tagsToRemove, currentList);
        
        itemToEdit.setName(new Name(firstTestName));
        itemToEdit.setStartDate(new ItemDate(validStartDate));
        currentList = TestUtil.replaceItemFromList(currentList, itemToEdit, targetIndex-1);
    
        //edit the last in the list which is an event
        targetIndex = currentList.length;
        itemToEdit = currentList[targetIndex-1];
        
        assert itemToEdit.getItemType().isEvent();
        
        assertEditSuccess(COMMAND_WORD, targetIndex, null, null, null, null, expectedNlpEndDate, 
                          expectedNlpEndTime, validNlpEndDateTime, tagsToAdd, null, currentList);
        
        assertEditSuccess(COMMAND_WORD, targetIndex, null, null, null, null, validEndDate, null, null, 
                          tagsToAdd, null, currentList);
        
        tagsToRemove = new UniqueTagList();
        tagsToRemove.add(new Tag("work"));
        
        assertEditSuccess(COMMAND_WORD, targetIndex, secondTestName, null, null, null,
                          null, null, null, tagsToAdd, tagsToRemove, currentList);
        
        tagsToAdd.add(new Tag("notplay"));
        
        assertEditSuccess(COMMAND_WORD, targetIndex, null, null, null, null, 
                          null, validEndTime, null, tagsToAdd, null, currentList);
        
        assertEditSuccess(SHORT_COMMAND_WORD, targetIndex, null, expectedNlpStartDate, expectedNlpStartTime,
                          validNlpStartDateTime, null, null, null, null, null, currentList);
        
        assertEditSuccess(SHORT_COMMAND_WORD, targetIndex, null, null, validStartTime, 
                          null, null, null, null, null, null, currentList);
        
        assertEditSuccess(SHORT_COMMAND_WORD, targetIndex, null, validStartDate, null, null, 
                          null, null, null, null, null, currentList);
        
        itemToEdit.setName(new Name(secondTestName));
        currentList = TestUtil.replaceItemFromList(currentList, itemToEdit, targetIndex-1);
        
        //edit from the middle of the list which is a task
        targetIndex = currentList.length/2 + 1;
        itemToEdit = currentList[targetIndex-1];
        
        assert itemToEdit.getItemType().isTask();
        
        assertEditSuccess(COMMAND_WORD, targetIndex, thirdTestName, null, null, null, null, 
                          null, null, tagsToAdd, null, currentList);
        
        assertEditSuccess(COMMAND_WORD, targetIndex, thirdTestName, null, null, null, validEndDate, 
                          validEndTime, null, tagsToAdd, null, currentList);
        
        assertEditSuccess(COMMAND_WORD, targetIndex, thirdTestName, null, null, null, expectedNlpEndDate, 
                          expectedNlpEndTime, validNlpEndDateTime, tagsToAdd, null, currentList);
        
        itemToEdit.setName(new Name(thirdTestName));
        itemToEdit.setItemType(new ItemType(ItemType.DEADLINE_WORD));
        itemToEdit.setEndDate(new ItemDate(expectedNlpEndDate));
        itemToEdit.setEndTime(new ItemTime(expectedNlpEndTime));
        currentList = TestUtil.replaceItemFromList(currentList, itemToEdit, targetIndex-1);

        //edit from the middle of the list which is a deadline
        targetIndex = 8;
        itemToEdit = currentList[targetIndex-1];
        
        assert itemToEdit.getItemType().isDeadline();
        
        assertEditSuccess(COMMAND_WORD, targetIndex, null, null, null, null, 
                          validEndDate, null, null, null, null, currentList);
        
        assertEditSuccess(SHORT_COMMAND_WORD, targetIndex, thirdTestName, null, null, null, 
                          null, null, null, tagsToAdd, null, currentList);
        
        tagsToRemove.add(new Tag("notwork"));
        
        assertEditSuccess(COMMAND_WORD, targetIndex, null, null, null, null, 
                          null, validEndTime, null, tagsToAdd, null, currentList);        
    }
    
    @Test
    public void edit_enterInvalidIndex_invalidCommandMessage() {
        commandBox.runCommand("edit notAnIndex");
        assertResultMessage(String.format(MESSAGE_INVALID_COMMAND_FORMAT, MESSAGE_USAGE));
    }
    
    @Test
    public void edit_enterInvalidCommandWord_unknownCommandMessage() {
        commandBox.runCommand("edits 1");
        assertResultMessage(MESSAGE_UNKNOWN_COMMAND);
    }
        
    @Test
    public void edit_noParameterSpecified_invalidCommandMessage() {
        commandBox.runCommand("edit 1");
        assertResultMessage(String.format(MESSAGE_INVALID_COMMAND_FORMAT, MESSAGE_USAGE));
        
    }
    
    @Test
    public void editTask_invalidParameterSpecified_invalidCommandMessage() {
        TestItem[] currentList = td.getTypicalItems();
        commandBox.runCommand("edit " + (currentList.length/2 + 1) + " st/" + validStartTime);
        assertResultMessage(String.format(MESSAGE_INVALID_COMMAND_FORMAT, MESSAGE_USAGE));
        
        commandBox.runCommand("edit " + (currentList.length/2 + 1) + " sd/" + validStartDate);
        assertResultMessage(String.format(MESSAGE_INVALID_COMMAND_FORMAT, MESSAGE_USAGE));
    }
    
    @Test
    public void editTask_dateNotSpecified_conversionFailedMessage() {
        TestItem[] currentList = td.getTypicalItems();
        commandBox.runCommand("edit " + (currentList.length/2 + 1) + " sd/" + validStartDate);
        assertResultMessage(String.format(MESSAGE_INVALID_COMMAND_FORMAT, MESSAGE_USAGE));
    }
    
    @Test
    public void editTask_timeNotSpecified_conversionFailedMessage() {
        TestItem[] currentList = td.getTypicalItems();
        commandBox.runCommand("edit " + (currentList.length/2 + 1) + " et/" + validEndTime);
        assertResultMessage(UnableToConvertFromTaskToDeadlineException.MESSAGE_CONVERSION_FAILED);
    }
    
    @Test
    public void edit_addDuplicateTag_duplicateItemMessage() {
        commandBox.runCommand("edit 2 #work");
        assertResultMessage(MESSAGE_NOTHING_EDITED);
    }
    
    @Test
    public void edit_deleteNonExistentTag_TagNotFoundMessage() {
        commandBox.runCommand("edit 2 #-" + nonexistentTag );
        assertResultMessage(String.format(MESSAGE_INVALID_COMMAND_FORMAT, 
                            String.format(TagNotFoundException.MESSAGE_TAG_NOT_FOUND, 
                                          "[" + nonexistentTag + "]")));
    }
       
    @Test
    public void edit_invalidIndexSpecified_invalidIndexMessage() {
        TestItem[] currentList = td.getTypicalItems();
        commandBox.runCommand("edit " + currentList.length + 1 + " n/" + firstTestName);
        assertResultMessage(MESSAGE_INVALID_ITEM_DISPLAYED_INDEX);
    }
    
    @Test
    public void edit_invalidTimeFormat_timeConstraintsMessage() {
        TestItem[] currentList = td.getTypicalItems();
        commandBox.runCommand("edit " + currentList.length + " et/" + invalidTime);
        assertResultMessage(MESSAGE_TIME_CONSTRAINTS);
    }

    @Test
    public void edit_invalidDateFormat_dateConstraintsMessage() {
        TestItem[] currentList = td.getTypicalItems();
        commandBox.runCommand("edit " + currentList.length + " ed/" + invalidDate);
        assertResultMessage(MESSAGE_DATE_CONSTRAINTS);
    }
    
    @Test
    public void edit_endDateComesBeforeStartDate_endDateTimeBeforeStartDateTimeMessage() {
        TestItem[] currentList = td.getTypicalItems();
        
        commandBox.runCommand("edit " + currentList.length + " ed/" + invalidOrderEndDate + " sd/" 
                              + invalidOrderStartDate);
        
        assertResultMessage(String.format(MESSAGE_INVALID_COMMAND_FORMAT, 
                                          MESSAGE_END_DATE_TIME_BEFORE_START_DATE_TIME));
    }
    
    @Test
    public void edit_endTimeComesBeforeStartTimeOnSameStartEndDate_endDateTimeBeforeStartDateTimeMessage() {
        TestItem[] currentList = td.getTypicalItems();
        
        commandBox.runCommand("edit " + currentList.length + " ed/" + validStartDate + " sd/" 
                              + validStartDate + " st/" + validStartTime + " et/" + validEndTime);
        commandBox.runCommand("edit " + currentList.length + " et/" + invalidOrderEndTime+ " st/" 
                              + invalidOrderStartTime);
        
        assertResultMessage(String.format(MESSAGE_INVALID_COMMAND_FORMAT, 
                                          MESSAGE_END_DATE_TIME_BEFORE_START_DATE_TIME));
    }

    /**
     * Runs the edit command to edit the item at specified index and confirms the result is correct.
     * @param targetIndexOneIndexed e.g. to edit the first item in the list, 1 should be given as the target index.
     * @param nlpStartDateTime start datetime given in natural language
     * @param nlpEndDateTime end datetime given in natural language
     * @param tagsToAdd tags to be added to the item
     * @param tagsToRemove tags to be removed from the item
     * @param currentList A copy of the current list of items (before editing).
     */
    private void assertEditSuccess(String commandWord, int targetIndexOneIndexed, String name, 
                                   String startDate, String startTime, String nlpStartDateTime, 
                                   String endDate, String endTime, String nlpEndDateTime, 
                                   UniqueTagList tagsToAdd, UniqueTagList tagsToRemove, 
                                   final TestItem[] currentList) throws IllegalValueException {
        
        TestItem itemToEdit = editTestItem(currentList[targetIndexOneIndexed-1], name, startDate, 
                                           startTime, endDate, endTime, tagsToAdd, tagsToRemove);
        
        TestItem[] expectedList = TestUtil.replaceItemFromList(currentList, itemToEdit, targetIndexOneIndexed-1);
        
        final StringBuilder commandBuilder = generateEditCommandString(commandWord, targetIndexOneIndexed, 
                                                 name, startDate, startTime, nlpStartDateTime, endDate, 
                                                 endTime, nlpEndDateTime, tagsToAdd, tagsToRemove);
        
        logger.info(commandBuilder.toString());
        commandBox.runCommand(commandBuilder.toString());

        assertTrue(shortItemListPanel.isListMatching(expectedList));

        assertResultMessage(String.format(MESSAGE_EDIT_ITEM_SUCCESS, itemToEdit));
    }

    /**
     * Creates command string in the context of the edit command using the arguments passed in
     * @return string containing the full edit command to be passed into runCommand
     */
    private StringBuilder generateEditCommandString(String commandWord, int targetIndexOneIndexed, 
                              String name, String startDate, String startTime, String startDateTime, 
                              String endDate, String endTime, String endDateTime, UniqueTagList tagsToAdd, 
                              UniqueTagList tagsToRemove) {
        
        final StringBuilder commandBuilder = new StringBuilder();
        commandBuilder.append(commandWord + " " + targetIndexOneIndexed);
        
        if (isToBeEdited(name)) {
            commandBuilder.append(" n/")
                          .append(name);
        }
        if (isToBeEdited(startDate)) {
            commandBuilder.append(" sd/")
                          .append(startDate);
        }
        if (isToBeEdited(startTime)) {
            commandBuilder.append(" st/")
                          .append(startTime);
        }
        if (isToBeEdited(startDateTime)) {
            commandBuilder.append(" sdt/")
                          .append(startDateTime);
        }
        if (isToBeEdited(endDate)) {
            commandBuilder.append(" ed/")
                          .append(endDate);
        }
        if (isToBeEdited(endTime)) {
            commandBuilder.append(" et/")
                          .append(endTime);
        }
        if (isToBeEdited(endDateTime)) {
            commandBuilder.append(" edt/")
                          .append(endDateTime);
        }
        if (isToBeEdited(tagsToAdd)) {
            Set<Tag> tagsToAddSet = tagsToAdd.toSet();
            for (Tag tag : tagsToAddSet) {
                commandBuilder.append(" #")
                              .append(tag.tagName);
            }
        }
        if (isToBeEdited(tagsToRemove)) {
            Set<Tag> tagsToRemoveSet = tagsToRemove.toSet();
            for (Tag tag : tagsToRemoveSet) {
                commandBuilder.append(" #-")
                              .append(tag.tagName);

            }
        }
        return commandBuilder;
    }

    /*
     * Edits test item according to parameters specified
     */
    private TestItem editTestItem(TestItem itemToEdit, String name, String startDate, 
                String startTime, String endDate, String endTime, 
                UniqueTagList tagsToAdd, UniqueTagList tagsToRemove) throws IllegalValueException {
        
        if (isToBeEdited(name)) {
            itemToEdit.setName(new Name(name));
        }
        if (isToBeEdited(startDate)) {
            itemToEdit.setStartDate(new ItemDate(startDate));
        }
        if (isToBeEdited(startTime)) {
            itemToEdit.setStartTime(new ItemTime(startTime));
        }
        if (isToBeEdited(endDate)) {
            itemToEdit.setEndDate(new ItemDate(endDate));
        }
        if (isToBeEdited(endTime)) {
            itemToEdit.setEndTime(new ItemTime(endTime));
        }
        if (isToBeConvertedFromTaskToDeadline(itemToEdit, endDate, endTime)) {
            itemToEdit.setItemType(new ItemType(ItemType.DEADLINE_WORD));
        }
        if (isToBeEdited(tagsToAdd)) {
            UniqueTagList tagsToEdit = itemToEdit.getTags();
            tagsToEdit.mergeFrom(tagsToAdd);
            itemToEdit.setTags(tagsToEdit);
        }
        if (isToBeEdited(tagsToRemove)) {
            UniqueTagList tagsToEdit = itemToEdit.getTags();
            tagsToEdit.remove(tagsToRemove);
            itemToEdit.setTags(tagsToEdit);
        }
        return itemToEdit;
    }

    /**
     * Checks if the item is to be converted from a task into a deadline
     */
    private boolean isToBeConvertedFromTaskToDeadline(TestItem itemToEdit, String endDate, String endTime) {
        return itemToEdit.getItemType().isTask() && isToBeEdited(endDate) && isToBeEdited(endTime);
    }
    
    /**
     * Checks if argument is to be edited
     */
    private boolean isToBeEdited(Object argument) {
        return argument != null;
    }
}
```
###### \java\guitests\ListNotDoneCommandTest.java
``` java
public class ListNotDoneCommandTest extends TaskManagerGuiTest {
	
	@Test
	public void listNotDone_success() {
	    assertListNotDoneCommandSuccess(COMMAND_WORD);
        assertListNotDoneCommandSuccess(SHORT_COMMAND_WORD);
	}
	
	@Test
    public void listNotDone_invalidCommand_unknownCommandMessage() {
        commandBox.runCommand("listnotdones");
        assertResultMessage(Messages.MESSAGE_UNKNOWN_COMMAND);
    }
	
	
	private void assertListNotDoneCommandSuccess(String commandWord) {
        commandBox.runCommand(commandWord);
        assertResultMessage(MESSAGE_SUCCESS);
    }

}


```
###### \java\seedu\taskmanager\testutil\TypicalTestItems.java
``` java
    public TypicalTestItems() {
        try {
            event1 = new ItemBuilder().withItemType("event").withStartDate("2016-06-06")
                                      .withStartTime("05:00").withEndTime("12:01").withEndDate("2016-08-08")
                                      .withName("Game of Life").withTags("play", "forever").build();
            
            deadline1 = new ItemBuilder().withItemType("deadline").withStartDate("").withStartTime("")
                                         .withEndTime("01:01").withEndDate("2016-12-06")
                                         .withName("This is a deadline").withTags("work", "important")
                                         .build();
            
            task1 = new ItemBuilder().withItemType("task").withName("Win at Life").withStartDate("")
                                     .withStartTime("").withEndDate("").withEndTime("").build();
            
            event2 = new ItemBuilder().withItemType("event").withName("This is an event")
                                      .withStartDate("2015-01-01").withStartTime("00:00")
                                      .withEndDate("2015-01-01").withEndTime("23:59").build();
            
            deadline2 = new ItemBuilder().withItemType("deadline").withName("Pay my bills")
                                         .withStartDate("")
                                         .withStartTime("").withEndDate("2019-09-09").withEndTime("10:30")
                                         .build();
            
            task2 = new ItemBuilder().withItemType("task").withName("This is a task").withStartDate("")
                                     .withStartTime("").withEndDate("").withEndTime("").build();
            
            event3 = new ItemBuilder().withItemType("event").withName("2103 exam")
                                      .withStartDate("2016-01-01").withStartTime("13:59")
                                      .withEndDate("2017-01-03").withEndTime("15:00").build();
            
            deadline3 = new ItemBuilder().withItemType("deadline").withName("Submit report")
                                         .withStartDate("").withStartTime("").withEndDate("2016-09-30")
                                         .withEndTime("21:14").build();
            
            task3 = new ItemBuilder().withItemType("task").withName("Buy one dozen cartons of milk")
                                     .withStartDate("").withStartTime("").withEndDate("").withEndTime("")
                                     .build();
            
            event4 = new ItemBuilder().withItemType("event").withName("Java Workshop")
                                      .withStartDate("2017-01-02").withStartTime("08:00")
                                      .withEndDate("2017-01-02").withEndTime("12:00").withTags("important")
                                      .build();

            //Manually added
            deadline4 = new ItemBuilder().withItemType("deadline").withName("Submit essay assignment")
                                         .withStartDate("").withStartTime("").withEndDate("2016-11-28")
                                         .withEndTime("21:29").build();
            
            task4 = new ItemBuilder().withItemType("task").withName("Call for the electrician")
                                     .withStartDate("").withStartTime("").withEndDate("").withEndTime("")
                                     .build();
            
        } catch (IllegalValueException e) {
            e.printStackTrace();
            assert false : "not possible";
        }
    }

    public static void loadTaskManagerWithSampleData(TaskManager ab) {

        try {
            ab.addItem(new Item(event1));
            ab.addItem(new Item(deadline1));
            ab.addItem(new Item(task1));
            ab.addItem(new Item(event2));
            ab.addItem(new Item(deadline2));
            ab.addItem(new Item(task2));
            ab.addItem(new Item(event3));
            ab.addItem(new Item(deadline3));
            ab.addItem(new Item(task3));
            ab.addItem(new Item(event4));
//            ab.addItem(new Item(deadline4));
//            ab.addItem(new Item(task4));
        } catch (UniqueItemList.DuplicateItemException e) {
            assert false : "not possible";
        }
    }

    public TestItem[] getTypicalItems() {
        return new TestItem[]{event1, deadline1, task1, event2, deadline2, task2, event3, deadline3, task3,
                event4}; //deadline4, task4};
    }
    public TaskManager getTypicalTaskManager(){
        TaskManager ab = new TaskManager();
        loadTaskManagerWithSampleData(ab);
        return ab;
    }
}
```
