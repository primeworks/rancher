# rancher
My rancher YAML codes

This contains yaml launch codes for Odoo v.15 CE, Postgres 12, Nodeport ingress service and Persistent Volume Claim.

Updating these files once the application is running on K8s will automatically trigger updates on these microservices so tread softly.

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: odoo-eliteboxng
    tier: frontend
    cattle.io/creator: norman
    workload.user.cattle.io/workloadselector: deployment-default-odoo-eliteboxng
  name: odoo-eliteboxng
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      workload.user.cattle.io/workloadselector: deployment-default-odoo-eliteboxng
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      annotations:
        field.cattle.io/ports: '[[]]'
      creationTimestamp: null
      labels:
        workload.user.cattle.io/workloadselector: deployment-default-odoo-eliteboxng
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: stack
                operator: In
                values:
                - odoo
      containers:
      - env:
        - name: HOST
          value: db-eliteboxng
        - name: PASSWORD
          value: odoo
        - name: USER
          value: odoo
        image: odoo:13
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 6
          initialDelaySeconds: 60
          periodSeconds: 30
          successThreshold: 1
          tcpSocket:
            port: 8069
          timeoutSeconds: 5
        ports:
        - containerPort: 8069
          name: http
          protocol: TCP
        startupProbe:
            failureThreshold: 3
            successThreshold: 1
            initialDelaySeconds: 0
            timeoutSeconds: 1
            periodSeconds: 10
            exec:
              command:
                - /bin/sh
                - '-c'
                - '-e'
                - exec pg_isready -U "odoo" -d "dbname=postgres" -h db-eliteboxng
        readinessProbe:
          failureThreshold: 6
          initialDelaySeconds: 30
          periodSeconds: 10
          successThreshold: 1
          tcpSocket:
            port: 8069
          timeoutSeconds: 5
        name: odoo-eliteboxng
        resources:
          requests:
            cpu: 100m
            memory: 512Mi
        securityContext:
          allowPrivilegeEscalation: false
          capabilities: {}
          privileged: false
          readOnlyRootFilesystem: false
          runAsNonRoot: false
        stdin: true
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        tty: true
        ports:
            - containerPort: 8069
              name: http
              protocol: TCP
        volumeMounts:
        # Edit the volume to match the existing mountPaths in the cluster
        - mountPath: /var/lib/odoo
          name: vol-7xe3r
          subPath: odoo
        - mountPath: /mnt/extra-addons
          name: vol-7xe3r
          subPath: odoo/addons
        - mountPath: /etc/odoo
          name: config
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      tolerations:
      - effect: NoSchedule
        key: stack
        operator: Equal
        value: odoo
      volumes:
      - configMap:
          defaultMode: 509
          name: odoo.conf
          optional: true
        name: config
      # Edit the volume to match the existing pv in the cluster
      - name: vol-7xe3r
        persistentVolumeClaim:
          claimName: odoo-eliteboxng

