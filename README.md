# WebCalendarBackend

## Example of the EventController.class

```java

@RestController
public class EventController {

    private final EventRepository eventRepository;

    @Autowired
    public EventController(EventRepository eventRepository) {
        this.eventRepository = eventRepository;
    }

    @GetMapping("/event/today")
    public ResponseEntity<List<Event>> getTodayEvents() {
        LocalDate today = LocalDate.now();
        List<Event> todayEvents = eventRepository.findAll().stream()
                .filter(event -> event.getDate().isEqual(today))
                .collect(Collectors.toList());


        return ResponseEntity.ok(todayEvents);
    }

    @GetMapping("/event")
    public ResponseEntity<List<Event>> getAllEvents(
            @RequestParam(required = false) String start_time,
            @RequestParam(required = false) String end_time) {
        if (start_time == null || end_time == null) {
            List<Event> allEvents = eventRepository.findAll();
            if (allEvents.isEmpty()) {
                return ResponseEntity.noContent().build();
            }
            return ResponseEntity.ok(allEvents);
        }

        LocalDate startDate = LocalDate.parse(start_time);
        LocalDate endDate = LocalDate.parse(end_time);

        List<Event> filteredEvents = eventRepository.findAll().stream()
                .filter(event -> !event.getDate().isBefore(startDate) && !event.getDate().isAfter(endDate))
                .collect(Collectors.toList());

        if (filteredEvents.isEmpty()) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.ok(filteredEvents);
    }

    @PostMapping("/event")
    public ResponseEntity<Map<String, Object>> createEvent(@Valid @RequestBody EventRequest eventRequest) {
        if (!isValidEvent(eventRequest)) {
            return ResponseEntity.badRequest().build();
        }
        Event event = new Event();
        event.setEvent(eventRequest.getEvent());

        // Parse the date string into a LocalDate object
        LocalDate date = LocalDate.parse(eventRequest.getDate());
        event.setDate(date);

        // Save the event to the database
        event = eventRepository.save(event);

        // Prepare the response body
        Map<String, Object> responseBody = new HashMap<>();
        responseBody.put("message", "The event has been added!");
        responseBody.put("event", event.getEvent());
        responseBody.put("date", event.getDate().toString());

        return ResponseEntity.ok(responseBody);
    }

    @GetMapping("/event/{id}")
    public ResponseEntity<?> getEventById(@PathVariable int id) {
        Optional<Event> eventOptional = eventRepository.findById((long) id);
        if (eventOptional.isPresent()) {
            Event event = eventOptional.get();
            Map<String, Object> responseBody = new HashMap<>();
            responseBody.put("id", event.getId());
            responseBody.put("event", event.getEvent());
            responseBody.put("date", event.getDate().toString());
            return ResponseEntity.ok(responseBody);
        }
        Map<String, String> errorResponse = new HashMap<>();
        errorResponse.put("message", "The event doesn't exist!");
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
    }

    @DeleteMapping("/event/{id}")
    public ResponseEntity<?> deleteEvent(@PathVariable int id) {
        Optional<Event> eventOptional = eventRepository.findById((long) id);
        if (eventOptional.isPresent()) {
            Event deletedEvent = eventOptional.get();
            eventRepository.delete(deletedEvent);
            Map<String, Object> responseBody = new HashMap<>();
            responseBody.put("id", deletedEvent.getId());
            responseBody.put("event", deletedEvent.getEvent());
            responseBody.put("date", deletedEvent.getDate().toString());
            return ResponseEntity.ok(responseBody);
        }
        Map<String, String> errorResponse = new HashMap<>();
        errorResponse.put("message", "The event doesn't exist!");
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Void> handleValidationExceptions(MethodArgumentNotValidException ex) {
        return new ResponseEntity<>(HttpStatus.BAD_REQUEST);
    }

    private boolean isValidEvent(EventRequest eventRequest) {
        if (eventRequest.getEvent() == null || eventRequest.getEvent().trim().isEmpty()) {
            return false;
        }
        if (eventRequest.getDate() == null || !eventRequest.getDate().matches("\\d{4}-\\d{2}-\\d{2}")){
            return false;
        }
        try {
            LocalDate.parse(eventRequest.getDate());
        } catch (Exception e) {
            return false;
        }
        return true;
    }
}
```
 
