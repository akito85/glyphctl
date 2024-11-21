**Key Properties of a Container:**

1. **name:**
    
    - A unique name for the container.
    - **Example:**
        
        YAML
        
        ```
        name: my-container
        ```
        
        Use code [with caution.](/faq#coding)
        
2. **image:**
    
    - The Docker image to use for the container.
    - **Example:**
        
        YAML
        
        ```
        image: my-image:latest
        ```
        
        Use code [with caution.](/faq#coding)
        
3. **command:**
    
    - The command to execute when the container starts.
    - **Example:**
        
        YAML
        
        ```
        command: ["python", "-m", "app"]
        ```
        
        Use code [with caution.](/faq#coding)
        
4. **args:**
    
    - Arguments to pass to the command.
    - **Example:**
        
        YAML
        
        ```
        args: ["--port", "8080"]
        ```
        
        Use code [with caution.](/faq#coding)
        
5. **ports:**
    
    - Defines the ports exposed by the container.
    - **Example:**
        
        YAML
        
        ```
        ports:
        - containerPort: 8080
          protocol: TCP
        ```
        
        Use code [with caution.](/faq#coding)
        
6. **env:**
    
    - Sets environment variables for the container.
    - **Example:**
        
        YAML
        
        ```
        env:
        - name: MY_VAR
          value: "my-value"
        ```
        
        Use code [with caution.](/faq#coding)
        
7. **envFrom:**
    
    - Sources environment variables from ConfigMaps or Secrets.
    - **Example:**
        
        YAML
        
        ```
        envFrom:
        - configMapRef:
            name: my-config-map
        ```
        
        Use code [with caution.](/faq#coding)
        
8. **volumeMounts:**
    
    - Mounts volumes into the container.
    - **Example:**
        
        YAML
        
        ```
        volumeMounts:
        - name: my-volume
          mountPath: "/mnt/data"
        ```
        
        Use code [with caution.](/faq#coding)
        
9. **resources:**
    
    - Specifies the resource limits and requests for the container.
    - **Example:**
        
        YAML
        
        ```
        resources:
          limits:
            cpu: "2"
            memory: "2Gi"
          requests:
            cpu: "1"
            memory: "1Gi"
        ```
        
        Use code [with caution.](/faq#coding)
        
10. **securityContext:**
    

- Configures security options for the container.
- **Example:**
    
    YAML
    
    ```
    securityContext:
      capabilities:
        add: ["CAP_NET_ADMIN"]
      privileged: true
    ```
    
    Use code [with caution.](/faq#coding)
    

11. **lifecycle:**

- Defines lifecycle hooks for the container, such as preStop and postStart.
- **Example:**
    
    YAML
    
    ```
    lifecycle:
      postStart:
        exec:
          command: ["sh", "-c", "echo 'Container started'"]
    ```
    
    Use code [with caution.](/faq#coding)
    

12. **livenessProbe:**

- Defines a probe to check if the container is alive.
- **Example:**
    
    YAML
    
    ```
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 30
    ```
    
    Use code [with caution.](/faq#coding)
    

13. **readinessProbe:**

- Defines a probe to check if the container is ready to serve traffic.
- **Example:**
    
    YAML
    
    ```
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    ```
    
    Use code [with caution.](/faq#coding)
    

By customizing these properties, you can fine-tune the behavior and resource allocation of your containers within a Kubernetes Pod.