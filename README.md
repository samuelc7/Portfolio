# Portfolio
Detailed description about who I am and the projects that I have worked on throughout my coding career. 

# Who Am I
* I am a student at Brigham Young University-Idaho working torwards a Bachelors Degree in Computer Science. 

# Relevant Job Timeline / Projects
  * ## May, 2021 - Hired by BYU-I Support Center as a *Reports and Systems Developer*:
    *  ### Sample Project while working with BYU-I Support Center: 
       *  October, 2021 - Reengineer CleanUp project for indivdual cisco phones:
          * ### Background / Overview:
            At the BSC (BYU-I Support Center) we have a calls center where agents recieve phone calls from students at the university that need help. Our Job as part of the Business Solutions team in the BSC is to help keep track of all the calls we recieve so that we can see how we (The BSC) are doing in helping out all the students that call in. Some things that we track is the amount of phone calls each agent recieves, the amount of calls agents pick up, the length of each call and more all throughout each agent's shift. We use reports sent to us from [cisco](https://www.cisco.com) to get all of the data wanted for each agent which then we report in [TeamDynamix](https://www.teamdynamix.com).
            
            A Major Program which was used to retrieve and report all of the data of each call broke due to changes in excel formatting - we were getting incorrect data from excel when parsing each sheet to get the cell values.

            Because of this break I was put in charge of fixing the lost/incorrect data from the past and creating a new way of getting data from cisco. 

            ### Project Layout:
            My team and I decided that it was best to switch from recieving excel reports from [cisco](https://www.cisco.com) to recieving raw data (in html format). My coworkers Jacob and Nick were put in charge of parsing the html formatted reports into a data structure such that would provide easy access to agent data. 
            
            ### My Tasks: 
            1. Get Tickets that needed data correction:
                 * *Side Note - We use TeamDynamix's ticketing system to keep track of certain data like described here: [TeamDynamix Tickets](https://solutions.teamdynamix.com/TDClient/1965/Portal/KB/ArticleDet?ID=3574)
                 * Using [TeamDynamix's API](https://td.byui.edu.api.teamdynamix.com/TDWebApi/), find and grab data for all tickets that had missing information.
                 * Store all tickets needing correction by date to be able to search for cisco reports by date in the next steps of program. 
                 * Code Snippet for this task which shows the start of grabbing the tickets that need to be updated:
                      ```
                        private Map<String, List<Map<String, Integer>>> getTickets() throws TDException {
                            Report patchMeReport = new Report();
                            try {
                                patchMeReport = teamDynamix.getReport(PATCH_ME_REPORT_ID, true, "");
                            } catch (Exception e) {
                                e.printStackTrace();
                        }
                        List<PatchTicket> patchTicketList = new ArrayList<>();
                        for (Row row : patchMeReport) { 
                            ...
                            patchTicketList.add(new PatchTicket(dataDate, ticket));
                        }
                    ```

            2. Get correct cisco reports from [cisco](https://www.cisco.com):
                 * Use functionality written by coworker Jacob to get and parse cisco reports. 
                 * Validate that cisco reports received are the correct ones wanted. 
                 * Pair each ticket to corresponding cisco report.
            3. Get data for each agent:
                 * Put agent data in data structure where all of the data for each agent is stored by the ticket unique id.
                 * Using the [TeamDynamix's API](https://td.byui.edu.api.teamdynamix.com/TDWebApi/), collect other data points from TeamDynamix reports and add them to each of the agent attributes.
                 * Code Snippet for this task which shows adding some data points to the created data structure:
                 ```
                 ...
                 // add all data points from the agent state summary report to agentValues map
                        for (String dataPoint : TeamDynamixData.getStateReportDataPoints()) {
                            String value = agentStateReport.get(agentName, dataPoint);
                            if (agentValues.containsKey(ticketID)) {
                                agentValues.get(ticketID).put(dataPoint, value);
                            } else {
                                Map<String, String> map = new HashMap<>();
                                map.put(dataPoint, value);
                                agentValues.put(ticketID, map);
                            }
                        }
                        ...
                 ```
            4. Patch the Tickets with correct data found:
                 * Create a Json patch array with correct operations and data pulled from the agent data structure created in previous step. 
                 * Using the [TeamDynamix's API](https://td.byui.edu.api.teamdynamix.com/TDWebApi/), patch each ticket with the json patch array created for each agent. 
                 * Code snippet for this task which shows a little bit of patching each ticket:
                    ```
                    private void patchTicket(int ticketId, Map<String, String> agentValues) {
                        DecimalFormat df = new DecimalFormat("0.00");
                        JsonPatchArray jsonPatchArray = new JsonPatchArray();

                        jsonPatchArray.addPatchOperation("add", "attributes/"...);
                        jsonPatchArray.addPatchOperation("add", "attributes/"...);
                        try {
                            jsonPatchArray.addPatchOperation("add", "attributes/" + ...);
                            ...
                        } catch (NullPointerException e) {
                            System.out.println("Could not get % Not Ready / % Ready / % Talking for ticket: " + ticketId);
                        }
                        ...
                    ```
            ### What I learned:

            1. Spring Boot:
               * All of our major programs use [Spring Boot](https://spring.io/projects/spring-boot) so I have been learning a lot about the extension and how to use it. Spring boot has made building and running applications very simple which has been great for what we have needed at the office. Spring Boot provides a lot of services but the one that we use most is their REST services. [Building Rest Services with Spring](https://spring.io/guides/tutorials/rest/) was a great resource to learn more about Spring Rest Services. I mainly worked with the annotation @RestController which pretty much ensures that every request handling method returns Http Responses. 
            2. Gmail Api:
                * I also learned a lot about working with gmail's [api services](https://developers.google.com/gmail/api). [users.messages.get](https://developers.google.com/gmail/api/reference/rest/v1/users.messages/get) and [Gmail API](https://developers.google.com/resources/api-libraries/documentation/gmail/v1/java/latest/) were two very helpful resources in learning about how to retrieve specific messages and attachments from gmail. 
                * Gmail quickstart:
                  * Something that took a while for me to figure out was how to get started with working with gmail's api. This [Java Quickstart](https://developers.google.com/gmail/api/quickstart/java) link helped me get everything started becuase it provides a simple and generic 3 step proccess to build an run a sample Java command-line application that makes requests to the Gmail API. Here is what I learned and what I think will be useful for any future application that would require requests to the Gmail API:
                    * A credentials file needs to be created that stores the google authentication credentials that is retrieved after signing in with the account that is being used.
                    * A gmail service needs to be created by doing something like: 
                        ```
                        final NetHttpTransport HTTP_TRANSPORT = GoogleNetHttpTransportnewTrustedTransport();
                        Gmail service = new Gmail.Builder(HTTP_TRANSPORT, JSON_FACTORY, getCredentials(HTTP_TRANSPORT))
                            .setApplicationName(APPLICATION_NAME)
                            .build();
                        ```
                    * To get a list of messages from a gmail account you first have to create a ListMessagesResponse and then get the messages from the response which would look something like this: 
                        ```
                        ListMessagesResponse response = service.users().messages().list("me")
                            .setLabelIds(labelIds).setMaxResults(100L).execute();
                        // Create list of messages
                         List<Message> messages = response.getMessages();
                        ```
            3. Working with dates in Java:
                * Working with dates can be a pain in the programming world which is why I constantly found myself referring to W3Schools [Java Date and Time](https://www.w3schools.com/java/java_date.asp) page. Although working with dates in this project was annoying I do feel like I learned a lot about how to use them. I mainly worked with Java LocalDates, Dates, SimpleDateTimeFormatters, and DateTimeFormatters. 
  * ## December 2021 - Hired By [Westerwood Global](https://westerwoodglobal.com/usa/?sr=1) as a *Mantinence Technician*
    * ### Sample Project while working with BYU-I Support Center: 
      * Feburary, 2022 - Simple Web Page for daily tasks
        * ### Project Summary:
          * When working as a mantinence technician we would have daily morning tasks which had a lot of manual steps that I thought could be automated.
          * We would be given a list of machines/tools that had been worked on the day before which we would then manually look up where each one was located using an excel sheet. It was pretty tedious and took time to locate each and every machine. 
          * I took initiative and created a simple web page that had a text box where we could copy and past all of the machine/tool numbers in. I stored all of the machines and their locations in a map which the program would use to look up each of the entries inputed in the text box. The program outputed all of the locations of the machines grouped by bay area which saved us hours each week from having to manually find each machine location one at a time. 

