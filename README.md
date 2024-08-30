# TacheProjet

tacheprojet/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── tacheprojet/
│   │   │               ├── TacheProjetApplication.java  // Classe principale
│   │   │               ├── model/                       // Package pour les entités
│   │   │               │   └── Task.java                // Classe Task
│   │   │               ├── repository/                  // Package pour les repositories
│   │   │               │   └── TaskRepository.java      // Interface TaskRepository
│   │   │               ├── service/                     // Package pour les services
│   │   │               │   └── TaskService.java         // Classe TaskService
│   │   │               └── controller/                  // Package pour les contrôleurs
│   │   │                   └── TaskController.java      // Classe TaskController
│   └── test/
│       └── java/
│           └── com/
│               └── example/
│                   └── tacheprojet/
│                       ├── TacheProjetApplicationTests.java  // Tests d'intégration de base
│                       └── service/                         // Package pour les tests de service
│                           └── TaskServiceTest.java         // Test unitaire pour TaskService



Task.java

package com.example.tacheprojet.model;

import lombok.Data;
import javax.persistence.*;

@Entity
@Data
public class Task {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
    private String description;

    @Enumerated(EnumType.STRING)
    private Priority priority;

    public enum Priority {
        HIGH,
        MEDIUM,
        LOW
    }
}

TaskRepository.java

package com.example.tacheprojet.repository;

import com.example.tacheprojet.model.Task;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface TaskRepository extends JpaRepository<Task, Long> {
    List<Task> findAllByOrderByPriorityAsc();
}


TaskService.java

package com.example.tacheprojet.service;

import com.example.tacheprojet.model.Task;
import com.example.tacheprojet.repository.TaskRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class TaskService {

    private final TaskRepository taskRepository;

    public TaskService(TaskRepository taskRepository) {
        this.taskRepository = taskRepository;
    }

    public List<Task> getAllTasksSortedByPriority() {
        return taskRepository.findAllByOrderByPriorityAsc();
    }
}

TaskController.java

package com.example.tacheprojet.controller;

import com.example.tacheprojet.model.Task;
import com.example.tacheprojet.service.TaskService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/tasks")
public class TaskController {

    private final TaskService taskService;

    public TaskController(TaskService taskService) {
        this.taskService = taskService;
    }

    @GetMapping("/sorted-by-priority")
    public List<Task> getTasksSortedByPriority() {
        return taskService.getAllTasksSortedByPriority();
    }
}


//Correction de Bug

//Identifier le bug

public Task updateTask(Long id, Task updatedTask) {
    Task task = taskRepository.findById(id).orElseThrow(() -> new RuntimeException("Task not found"));
    
    if (updatedTask.getTitle() != null) {
        task.setTitle(updatedTask.getTitle());
    }
    
    // Bug potentiel : updatedTask.getPriority() peut être null
    task.setPriority(updatedTask.getPriority());
    
    return taskRepository.save(task);
}

// Correction du bug trouvé

public Task updateTask(Long id, Task updatedTask) {
    Task task = taskRepository.findById(id).orElseThrow(() -> new RuntimeException("Task not found"));

    if (updatedTask.getTitle() != null) {
        task.setTitle(updatedTask.getTitle());
    }

    if (updatedTask.getPriority() != null) {
        task.setPriority(updatedTask.getPriority());
    }
    
    return taskRepository.save(task);
}

// Le Test unitaire pour vérifier la correction du bug

package com.example.tacheprojet.service;

import com.example.tacheprojet.model.Task;
import com.example.tacheprojet.repository.TaskRepository;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

class TaskServiceTest {

    private TaskRepository taskRepository = Mockito.mock(TaskRepository.class);
    private TaskService taskService = new TaskService(taskRepository);

    @Test
    void testUpdateTaskWithNullPriority() {
        Task existingTask = new Task();
        existingTask.setId(1L);
        existingTask.setTitle("Existing Task");
        existingTask.setPriority(Task.Priority.MEDIUM);

        Task updatedTask = new Task();
        updatedTask.setTitle("Updated Task");
        // No priority set, simulating null

        Mockito.when(taskRepository.findById(1L)).thenReturn(Optional.of(existingTask));
        Mockito.when(taskRepository.save(existingTask)).thenReturn(existingTask);

        Task result = taskService.updateTask(1L, updatedTask);

        assertEquals("Updated Task", result.getTitle());
        assertEquals(Task.Priority.MEDIUM, result.getPriority()); // Priority should remain unchanged
    }
}

